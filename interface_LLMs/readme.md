# Crowd monitoring query system

## Overview

The **Crowd monitoring query system** is a FastAPI-based application that allows users to query room occupancy records based on specific criteria using an instruction-tuned language model. This system integrates a MongoDB database to store and retrieve occupancy data, while leveraging a pre-trained language model to generate natural language responses based on the queried data.

## Key Features

- **Query Occupancy**: Post room occupancy queries using `time` and `room_id` as criteria, and retrieve a natural language response generated by the instruction-tuned language model.
- **Language Model**: Uses a locally hosted language model (e.g., GPT-2 or others) to generate responses in a human-friendly way.
- **Database Integration**: Connects to a MongoDB collection to fetch and filter occupancy data.
- **Health Check**: API health check to ensure the service is up and running.
- **Chat Interface**: A chat endpoint to test the language model with custom prompts.

## File Structure

- **`llms.py`**: Contains the logic for loading the language model, generating responses, and querying occupancy records.
- **`main.py`**: Implements the FastAPI application, with endpoints for querying occupancy and generating responses from the language model.
- **`mongo_connector.py`**: Provides a connection to MongoDB and functionality to query occupancy records based on provided criteria.
- **`.env`**: Holds environment variables such as MongoDB connection URI, database name, collection name, and the language model path.
  
## Dependencies

Ensure you have the following installed:
- Python 3.12+
- `FastAPI` for building the API
- `pydantic` for data validation
- `torch` for PyTorch
- `transformers` for using the pre-trained language models
- `pymongo` for MongoDB integration
- `dotenv` for loading environment variables
- `unsloth` for loading the models
- `langchain` for interfacing with databases

1. **Create a Conda environment and activate it**:

   ```bash
   conda env create -f environment.yml
   conda activate llms
   ```

<font color="red">In case you are not using conda to setup your environment and want to use PIP please follow the unsloth documentation https://docs.unsloth.ai/get-started/installation/pip-install 
and install the required dependencies as follows:
</font>

```Python
!pip install "unsloth[colab-new] @ git+https://github.com/unslothai/unsloth.git"
!pip install --no-deps xformers trl peft accelerate bitsandbytes
```

2. **Configure environment variables**:

   Create a `.env` file at the root of the project and add the following variables:

   ```
   MONGO_URI=<Your MongoDB URI>
   DB=<Your MongoDB Database Name>
   COLLECTION=<Your MongoDB Collection Name>
   MODEL=<Path or name of the pre-trained language model>
   HF_TOKEN=<Your Hugging Face token if needed>
   ```

3. **Run the FastAPI server**:

   ```bash
   fastapi dev main.py
   ```

4. **Configure environment variables**:

   Create a `.env` file at the root of the project and add the following variables:

   ```
   MONGO_URI=<Your MongoDB URI>
   DB=<Your MongoDB Database Name>
   COLLECTION=<Your MongoDB Collection Name>
   MODEL=<Path or name of the pre-trained language model>
   HF_TOKEN=<Your Hugging Face token if needed> // if you are wants to work with unmodified LLAMA3.1 models
   ```

5. **Important**: Ensure you have CUDA installed if you wish to run the model on a GPU.

6. If you are working on the LLAMA models please use below link to get an HF_token: 
https://huggingface.co/settings/tokens

### Steps to Get a Hugging Face Token

  1. **Create a Hugging Face Account**:
    - Go to [Hugging Face](https://huggingface.co/join) and sign up for a new account or log in if you already have one.

  2. **Navigate to the Settings Page**:
    - Once logged in, click on your profile picture in the top right corner.
    - Select "Settings" from the dropdown menu.

  3. **Access the Tokens Section**:
    - In the settings page, find and click on the "Access Tokens" tab on the left sidebar.

  4. **Generate a New Token**:
    - Click on the "New token" button.
    - Provide a name for your token and select the appropriate role (e.g., `read`).
    - Click on the "Generate" button.

  5. **Copy the Token**:
    - Once the token is generated, copy it and store it securely.
    - You will need this token to access Hugging Face models and APIs.

  6. **Add the Token to Your Environment Variables**:
    - Open your `.env` file in the root of your project.
    - Add the following line, replacing `<Your Hugging Face Token>` with the token you just generated:
      ```
      HF_TOKEN=<Your Hugging Face Token>
     ```



## Minimum System Requirements

To run the **Occupancy Query Service** API with the language model, the following hardware specifications are recommended:

- **CPU**: Intel Core i5 or equivalent
- **RAM**: 16 GB
- **GPU**: CUDA-compatible GPU (required for running larger models efficiently)
  - **For 3.5B parameter models**: Minimum 4 GB VRAM
  - **For 8B parameter models**: Minimum 6 GB VRAM

**Note**: If you don't have access to a compatible GPU, the API can still run on CPU, but with significantly slower inference times for larger models. 
please refer to https://huggingface.co/docs/transformers/main/en/main_classes/quantization#offload-between-cpu-and-gpu

---

## API Endpoints

### 1. `/query_occupancy` (POST)
Query the occupancy database and receive a natural language response.

- **Request Body**:
  ```json
  {
      "time": "2023-09-21T15:00:00",
      "room_id": 101
  }
  ```

- **Response**:
  ```json
  {
      "response": "The occupancy records are [details of the records]."
  }
  ```

### 2. `/` (GET)
Returns a welcome message.

- **Response**:
  ```json
  {
      "message": "Welcome to the occupancy query service!"
  }
  ```

### 3. `/health` (GET)
Returns the health status of the API.

- **Response**:
  ```json
  {
      "status": "healthy"
  }
  ```

### 4. `/chat` (GET)
Chat with the language model by passing a custom prompt.

- **Query Parameter**:
  `prompt`: String prompt to be processed by the model.

- **Response**:
  ```json
  {
      "message": "Response from the language model based on the prompt."
  }
  ```

## How it Works

1. **Occupancy Querying**: The service accepts a query in the form of `time` and `room_id` (optional). It retrieves matching records from MongoDB and passes the records as a prompt to the language model to generate a human-readable response.
   
2. **Language Model**: A pre-trained language model (loaded using `FastLanguageModel`) is used for inference. It generates a response based on the occupancy records and any additional prompts.

3. **Garbage Collection**: To optimize GPU memory usage, garbage collection and GPU memory management are handled using PyTorch’s `empty_cache` and `ipc_collect` methods.

## Model Configuration

- **Quantization**: The model is loaded with 4-bit quantization (set by `load_in_4bit=True`) to reduce memory usage.
- **Max Sequence Length**: The maximum sequence length for the model is set to `2048`, and automatic scaling for RoPE (Rotary Positional Embedding) is enabled.
- **Model Customization**: You can replace the current model (e.g., GPT-2) with any compatible transformer model by setting the `MODEL` environment variable to the desired model name or path.


### To training your custom models please use the notebook from train_llms/Train_llms.ipynb