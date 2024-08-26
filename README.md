 
# ollama-custom-model
This repository provides scripts to convert, quantize, and deploy your fine-tuned models using Ollama. Convert models to GGUF format, optimize them for performance, and seamlessly integrate with Ollama on Linux. Includes tools for model conversion, quantization, and easy deployment with a Python API.

# Ollama Installation and Model Conversion Guide (Linux)

This guide provides step-by-step instructions to download and install Ollama on Linux, convert models to GGUF format, and use Ollama for serving models.

## 1. Download and Install Ollama on Linux

### 1.1 Download the Ollama Installation Script
```bash
curl -fsSL https://ollama.com/install.sh -o ollama-linux.sh
```

### 1.2 Make the Script Executable
```bash
chmod +x ollama-linux.sh
```

### 1.3 Run the Installation Script
```bash
sudo ./ollama-linux.sh
```

## 2. Initialize Environment Variables (Optional)
You can set these environment variables, though they have default values if not set:
```bash
export OLLAMA_MODELS=/path/to/output/model/
export OLLAMA_HOST="0.0.0.0:7000"  # Adjust if you want to specify a custom port or IP
```

## 3. Start Ollama Server
```bash
ollama serve
```
> **Tip**: Run `ollama serve` in a separate terminal or use tools like `tmux` to keep it running in the background.

## 4. create model image

### 4.1. pull a model form ollama repo
```bash
ollama pull <model_name>
```
search model name in ollama website

### 4.2. Create a self Model for Ollama
```bash
ollama create <your_model_name> -f <path_to_Modelfile/Modelfile>
```

## 5. Check Active Models
```bash
ollama list
```
Expected output:
```plaintext
NAME             ID                 SIZE   MODIFIED   
<your_model_name>:latest  365c0bd3c000   4.7 GB  2 days ago
```

## 6. Use the Model in a Python Script
Here's an example of how to use the model with a Python script:
```python
import requests

def get_ollama_response(model: str, prompt: str, stream: bool = False, url: str = "http://0.0.0.0:7000/api/chat"):
    payload = {
        "model": model,
        "messages": [{"role": "user", "content": prompt}],
        "stream": stream
    }

    try:
        response = requests.post(url, json=payload)
        response.raise_for_status()
        return response.json().get("message", {}).get("content", "")
    except requests.exceptions.RequestException as e:
        return f"An error occurred: {e}"

# Example usage
model_name = "your_model_name"  # Replace with the correct model name
prompt_text = "How are you?"
res = get_ollama_response(model_name, prompt_text)
print(res)
```

## 7. Steps to Convert a Model to GGUF Format

### 7.1 Clone the llama.cpp Repository
```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
```

### 7.2 Install Required Dependencies
```bash
pip install -r requirements.txt
```
```bash
pip install sentencepiece # if not yet installed
```

### 7.3 Convert the Hugging Face Format to GGUF Format
Make sure you are still in the `llama.cpp` directory.
```bash
python convert-hf-to-gguf.py /path/to/your/model/directory
```
Your file should end with `"F16.gguf"`.

## 8. Quantize the Model from 16-bit to 4-bit

### 8.1 Compile the Project in llama.cpp
```bash
make
```

### 8.2 Quantize the Model
```bash
./llama-quantize /path/to/original/model/model_name_F16.gguf /path/to/output/model/quantize_model_name_Q4_K_M.gguf Q4_K_M
```

## 9. Create a `Modelfile`

```bash
nano Modelfile
```
Update the `Modelfile` with the following content:

```plaintext
FROM /path/to/output/model/quantize_model_name_Q4_K_M.gguf
PARAMETER temperature 0.7
PARAMETER stop ""
PARAMETER stop ""
TEMPLATE """
system
{{ .System }}
user
{{ .Prompt }}
assistant
"""
SYSTEM """You are a helpful assistant."""
```
> **Note**: Ensure you replace the `FROM` location with the correct path to your `.gguf` model directory before saving the file.

## 10. Final Files in the Model Directory
After quantization, your model directory should contain:
- `/path/to/output/model/quantize_model_name_Q4_K_M.gguf`
- `/path/to/output/model/Modelfile`

**Start from step 4 if you have already run the Ollama server. If not, start from step 2.**

---

This guide provides a complete walkthrough from installing Ollama on Linux to converting and serving models in GGUF format. Follow each step carefully, and refer to the [Ollama website](https://www.ollama.com/) for any additional details or updates.

Prepared by: Faisal Fauzi
 
