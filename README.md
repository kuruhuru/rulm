## Training

Overview:

* Install dependencies. You will need Git LFS to download the model and a correct combination of the versions of `transformers`, `peft`, and `bitsandbytes`.
* Download a base model that you will be finetuning, for example, [huggyllama/llama-7b](https://huggingface.co/huggyllama/llama-7b).
* Fix treatment of `pad`, `bos`, `eos` tokens.
* Prepare your data as two JSONL files, with three fields for the `"instruct"` mode: `"instruction"`, `"input"`, `"output"`. Or the following fields for the `"chat"` mode: `"messages"`.
* Run training.

### Install libraries
```
sudo apt-get install git-lfs
pip install -r ../requirements.txt
```

### Download base model
```
git clone https://huggingface.co/huggyllama/llama-7b models/llama-7b
```

### Fix tokenizer
Edit the following files under `models/llama-7b`:

`tokenizer_config.json`:

```
{
    "tokenizer_class": "LlamaTokenizer",
    "model_max_length": 2048,
    "padding_side": "left",
    "bos_token": "<s>",
    "eos_token": "</s>",
    "unk_token": "<unk>",
    "clean_up_tokenization_spaces": false,
    "special_tokens_map_file": "special_tokens_map.json"  
}
```

`special_tokens_map.json`:

```
{
    "bos_token": "<s>",
    "eos_token": "</s>",
    "pad_token": "<unk>",
    "sep_token": "<s>",
    "unk_token": "<unk>"
}
```

`generation_config.json`:

```
{
  "_from_model_config": true,
  "pad_token_id": 0,
  "bos_token_id": 1,
  "eos_token_id": 2
}
```

### Prepare data

Create two JSONL files with training and validation sets. See [create_instruct_set.py](https://github.com/IlyaGusev/rulm/blob/master/self_instruct/src/data_processing/create_instruct_set.py) or [create_char_set.py](https://github.com/IlyaGusev/rulm/blob/master/self_instruct/src/data_processing/create_char_set.py) for an example.

### Run training
```python
python3 -m src.train --config-file configs/saiga_7b.json --train-file train.jsonl --val-file val.jsonl  --output-dir models/saiga_7b
```
