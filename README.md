# Beyond Language Models: Byte Models are Digital World Simulators

This repository contains the code for the bGPT model as described in the paper [Beyond Language Models: Byte Models are Digital World Simulators](https://arxiv.org/pdf/%3CARXIV%20PAPER%20ID%3E.pdf).

You can check out the [demo page](https://byte-gpt.github.io/), which includes examples generated by the bGPT model.

## Model Description
Traditional deep learning often overlooks bytes, the basic units of the digital world, where all forms of information and operations are encoded and manipulated in binary format. Inspired by the success of next token prediction in natural language processing, we introduce bGPT, a model with next byte prediction to simulate the digital world. bGPT matches specialized models in performance across various modalities, including text, audio, and images, and offers new possibilities for predicting, simulating, and diagnosing algorithm or hardware behaviour. It has almost flawlessly replicated the process of converting symbolic music data, achieving a low error rate of 0.0011 bits per byte in converting ABC notation to MIDI format. In addition, bGPT demonstrates exceptional capabilities in simulating CPU behaviour, with an accuracy exceeding 99.99% in executing various operations. Leveraging next byte prediction, models like bGPT can directly learn from vast binary data, effectively simulating the intricate patterns of the digital world.

We provide five weights of bGPT on [Hugging Face](https://huggingface.co/sander-wood/bgpt) corresponding to each dataset used for pre-training:

1. **_weights-conversion.pth_**: bGPT pre-trained on IrishMAN for data conversion.
2. **_weights-cpu.pth_**: bGPT pre-trained on CPU states for CPU state modelling.
3. **_weights-text.pth_**: bGPT pre-trained on Wikipedia for text generation/classification.
4. **_weights-image.pth_**: bGPT pre-trained on ImageNet for image generation/classification.
5. **_weights-audio.pth_**: bGPT pre-trained on Librispeech for audio generation/classification.

## Installation

To set up the bGPT environment and install the necessary dependencies, follow these steps:

1. **Create and Activate Conda Environment**

   ```bash
   conda create --name bgpt python=3.7.9
   conda activate bgpt
   ```

2. **Install Dependencies**

   ```bash
   pip install -r requirements.txt
   ```

3. **Download Pre-trained bGPT Weights (Optional)**

   ```bash
   pip install torch==1.13.1+cu116 torchvision==0.14.1+cu116 torchaudio==0.13.1 --extra-index-url https://download.pytorch.org/whl/cu116
   ```
4. Download Pre-trained bGPT Weights (Optional)
   
   For those interested in starting with pre-trained models, bGPT weights are available on [Hugging Face](https://huggingface.co/sander-wood/bgpt). This step is optional but recommended for users looking to leverage the model's capabilities without training from scratch.
   
## Usage

- `config.py`: Configuration settings for training and inference.
- `cpu-simulation.py`: Simulate CPU states and operations.
- `inference.py`: Perform inference tasks (e.g., generation and conversion) using pre-trained models.
- `train-cls.py`: Training script for classification models.
- `train-gen.py`: Training script for generative models.
- `utils.py`: Utility functions supporting model operations and data processing.
  
### Configuration
The `config.py` file contains critical settings for training and inference, allowing for flexibility across different tasks and modalities. Here's a breakdown of the main configurations:

#### Training Configuration

- **TRAIN_FOLDERS**: Specify the dataset folders for training. Multiple folders can be included.
- **EVAL_FOLDERS**: Specify evaluation dataset folders.
- **PRE_WEIGHTS_PATH**: Path to pre-trained weights for transfer learning and fine-tuning.
- **WEIGHTS_PATH & LOGS_PATH**: Define locations to save trained weights and logs, respectively.
- **NUM_EPOCHS, LEARNING_RATE, BATCH_SIZE**: Control training duration, learning rate, and batch size for optimal learning.
- **ACCUMULATION_STEPS**: Set accumulation steps to emulate larger batch sizes, managing memory usage efficiently.
- **PATCH_SAMPLING_BATCH_SIZE**: Adjust batch size for patch sampling during training to reduce computational load, with `0` for full batch processing.

#### Inference Configuration

- **INFERENCE_WEIGHTS_PATH**: Path to weights for inference.
- **MODE**: Determines operation mode (`convert` or `generate`), guiding the model for specific outcomes.
- **NUM_SAMPLES, TOP_K, TOP_P, TEMPERATURE**: Set sampling strategy during inference to control the diversity of outputs.

### Generative Modelling

Generative modelling with bGPT is a flexible and powerful approach to learning and generating new data across various formats. bGPT segments byte sequences into patches, predicts next patch features with a patch-level decoder, and reconstructs bytes within patches using these features with a byte-level decoder. The training for bGPT primarily revolves around generative modelling through next byte prediction as its core focus, playing a pivotal role in predicting and generating bytes. Here's how to get started:

1. **Prepare Your Data**: Since bGPT models information at the byte level, it can work with any type of file that exists on a computer, regardless of format. This means your training and evaluation datasets can include text, images, audio, or any other file type. Ensure that your datasets are ready and accessible for the model to train on and evaluate against.

2. **Adjust Configuration Settings**: Modify the `config.py` file to tailor the training process to your needs. At a minimum, you should update the `TRAIN_FOLDERS` and `EVAL_FOLDERS` to point to your actual data directories. Also, specify where to save the trained model weights and logs by setting `WEIGHTS_PATH` and `LOGS_PATH`. You may adjust other parameters based on your specific requirements. For instance, with the default `PATCH_SIZE=16` and `PATCH_LENGTH=512`, bGPT can model byte sequences up to 8KB. If your training files are larger, and you have sufficient computational resources, consider increasing these parameters to accommodate the larger file sizes.

3. **Leverage Pre-trained Weights (Optional)**: If you wish to fine-tune a pre-trained bGPT model, set `PRE_WEIGHTS_PATH` to the location of the pre-trained weights and ensure `LOAD_FROM_PRE_CHECKPOINT=True`. To train a model from scratch, simply set `LOAD_FROM_PRE_CHECKPOINT=False`.

4. **Start Training**: Run `train-gen.py` to begin the training process. The script will use the configurations set in `config.py` and apply the training data to learn generative models capable of producing new, unseen outputs in the format of your training data.

### Classification

Classification with bGPT leverages the model's ability to understand and differentiate between various types of data at a fundamental level. This involves extracting a global feature from the byte sequence, which is then processed by a classification head. Here's how to approach classification tasks with bGPT:

1. **Prepare Labelled Data**: Ensure your dataset consists of labelled data, which can be a mix of different formats. The model distinguishes between data types using the naming convention `label.ext`, where the label is derived from the filename, specifically `filename.split('_')[0]`. This means that the label for classification should be clearly reflected in the file name, such as "Business_1.txt". It is crucial to organize your files accordingly to facilitate accurate classification.

2. **Generative Modelling Before Classification (Strongly Recommended)**: Before embarking on classification tasks, it is highly recommended to perform generative modelling on the same dataset. Starting with weights trained through generative modelling provides a solid foundation for further fine-tuning in classification tasks. To do this, set `PRE_WEIGHTS_PATH` to your generative model weights and ensure `LOAD_FROM_PRE_CHECKPOINT=True`. Directly training a classification model from scratch without this pre-training step has been observed to result in significantly poorer performance. When fine-tuning for classification, ensure that `WEIGHTS_PATH` and `LOGS_PATH` are set to different locations to prevent overwriting previous models. Note that the classification model will inherit the bGPT's patch-level decoder and discard the byte-level decoder, so it's essential to keep the model parameters unchanged during this phase.

3. **Start Training for Classification**: Run `train-cls.py` to begin the classification training process. The script will utilize the previously set configurations and apply them to your labelled dataset. The model will learn to classify the input data into the defined categories based on the labels extracted from the filenames.

### Inference

Inference with bGPT allows you to apply trained models to new data, performing tasks like data conversion or content generation based on the configurations set in `config.py`. Here's how to conduct inference using the provided settings:

1. **Set Up the Inference Configuration**: First, ensure your `config.py` includes the configuration for inference as shown above. Adjust the parameters according to the task you want to perform:

   - `INFERENCE_WEIGHTS_PATH`: This should point to the location of your trained model weights that you intend to use for inference. For example, `"weights-conversion"` indicates the model trained for converting files from one format to another.
   - `INPUT_EXT` and `TARGET_EXT`: These parameters define the extensions of the input and target files, respectively. In the given configuration, the model expects input files with the `.mid` extension and will aim to convert them into files with the `.abc` extension.
   - `MODE`: Determines the mode of inference. `"convert"` mode is used for converting files from one format to another, while `"generate"` mode is used for generating new content.
   - `NUM_SAMPLES`, `TOP_K`, `TOP_P`, and `TEMPERATURE`: These parameters control the sampling strategy for generation tasks, influencing the diversity and creativity of the output.

2. **Performing Conversion or Generation**: Depending on the `MODE` you've set, the inference process will either convert input files to a new format or generate new content:
   
   - **Conversion**: In `"convert"` mode, ensure your input files (e.g., `.mid`) are placed in a designated directory. The model will read these files, apply the conversion process, and output files in the target format (e.g., `.abc`) in the specified output directory.
   - **Generation**: In `"generate"` mode, the model will generate samples from scratch. The number of samples to generate is specified by `NUM_SAMPLES`. The generated samples will be placed in the output directory.

4. **Executing Inference**: To start the inference process, run the script `inference.py`. Make sure the script is configured to use the settings from `config.py` for conducting the desired inference task.

### CPU States Dataset

The CPU States Dataset is integral to exploring and modelling CPU behaviour, including a simplified yet detailed representation of CPU operations through sequences of machine instructions and subsequent register states. This dataset is crucial for training models like bGPT to predict how the CPU state updates with each instruction, demonstrating the model's proficiency in simulating digital processes within hardware environments.

The generation of the CPU States Dataset is facilitated by the `cpu-simulation.py` script, which allows for the generation, translation, and evaluation of CPU state sequences. The script operates in various modes to accommodate different stages of dataset preparation and model testing:

- **Generate Mode**: This mode is used to generate new instances of CPU states, simulating a wide range of program instructions to create a diverse dataset. It involves specifying the number of registers, memory size, and the range and quantity of instruction sequences to be generated. Example command:
  ```bash
  python cpu-simulation.py --mode generate --num_registers 10 --memory_size 1024 --dir_path cpu_states --num_min_program 1 --num_max_program 255 --num_train_instance 2100000 --num_test_instance 21000 --random_seed 0
  ```
  This generates training and testing instances with programs containing 1 to 255 instructions, tailored to the specified memory size and register count.

- **Translate Mode**: It is used to convert binary representations of CPU states into a human-readable format. This conversion is essential for manual inspection, debugging, and understanding the dataset's intricacies. Example command:
  ```bash
  python cpu-simulation.py --mode translate --states_path cpu_states/cpu.bin
  ```

- **Evaluate Mode**: To assess the accuracy of the model's predictions, the evaluate mode compares the predicted CPU states against the actual states within the dataset. Example command:
  ```bash
  python cpu-simulation.py --mode evaluate --dir_path cpu_states
  ```

The CPU States Dataset contains 2.1 million instances, each featuring a 1KB memory block and sequences of 16-byte CPU register states that include a variety of instruction types, such as data movement, logical, and arithmetic operations. This dataset simulates typical CPU behaviour, providing a rich resource for research and development in digital system simulation.

## BibTeX entry and citation info
```
```