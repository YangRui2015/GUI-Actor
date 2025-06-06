# Grounding Verifier for GUI-Actor

We developed a grounding verifier to assess whether a selected action position aligns with a given language instruction. This model is particularly effective for GUI-Actor, as GUI-Actor's attention map produces diverse candidate positions from a single inference. With the verifier, we can efficiently evaluate actions **in hindsight**—after identifying the chosen position on the image—and make more informed decisions.

<img src="https://cdn-uploads.huggingface.co/production/uploads/64d45451c34a346181b130dd/1LTBORYJsO9Ru6B4q_SKl.png" alt="image" width="500"/>

## Training

The verifier is trained to take a language instruction and an image (with a red circle marking the candidate position) as input, and predict whether the position is correct—outputting "True" or "False."

We use the [OS-Atlas dataset](https://huggingface.co/datasets/OS-Copilot/OS-Atlas-data) and process it using `verifier_data_generation.py` to curate training data. The model is fine-tuned via supervised training (SFT) starting from the UITARS-SFT-2B checkpoint, providing strong performance with a relatively small model size.

### Data Preparation

To prepare the dataset:

1. Download and unzip the [OS-Atlas dataset](https://huggingface.co/datasets/OS-Copilot/OS-Atlas-data) following the instructions on the Hugging Face page.
2. Organize the images into the following directory structure:

```python
image_folder_dict = {
    'windows_splited': f'{root_path}/desktop_domain/windows_images',
    'linux_splited': f'{root_path}/desktop_domain/linux_images',
    'macos_splited': f'{root_path}/desktop_domain/macos_images',
    'widget_captioning': f'{root_path}/mobile_domain/combined',
    'uibert_raw': f'{root_path}/mobile_domain/UIBert',
    'ricosca': f'{root_path}/mobile_domain/combined',
    'amex_raw': f'{root_path}/mobile_domain/amex_images',
    'seeclick_web': f'{root_path}/web_domain/seeclick_web_imgs',
    'fineweb_3m': f'{root_path}/web_domain/fineweb'
}
```

Each training sample includes a positive and one or more negative examples:

* **Positive samples**: taken directly from the original dataset with a red circle marking the correct target.
* **Negative samples**: created by either (a) selecting another meaningful UI element or (b) randomly sampling a point, which may not correspond to any actionable item.

To generate the dataset, run the following commands (since the dataset is very large, you can ):

```bash
python verifier_data_generation.py --root_path ${path_to_OS-Atlas-data} --new_directory ${save_path} --file_dict_key desktop_domain --selected_size 30000
python verifier_data_generation.py --root_path ${path_to_OS-Atlas-data} --new_directory ${save_path} --file_dict_key mobile_domain  --selected_size 30000
python verifier_data_generation.py --root_path ${path_to_OS-Atlas-data} --new_directory ${save_path} --file_dict_key web_domain     --selected_size 30000
```


### SFT 

We use the official code from [Aguvis](https://github.com/xlang-ai/aguvis) to perform SFT training. Make sure to set the file path correctly in the `stage1.yaml` configuration. For training, we use [**UITARS-2B-SFT**](https://huggingface.co/ByteDance-Seed/UI-TARS-2B-SFT) as the base model with a learning rate of $2 \times 10^{-5}$, running for one epoch.



## Evaluation

We evaluate our method using the attention weights generated by GUI-Actor and the grounding verifier, saved in a JSON file (e.g., `screenspot_all_preds_Original.json`). Before running the evaluation scripts, please update the file paths in `run_ss_v1.sh`, `run_ss_v2.sh`, and `run_ss_pro.sh` accordingly.

Make sure to download the ScreenSpot datasets and ensure their paths exactly match those specified in the shell scripts. Specifically, download **ScreenSpot** and **ScreenSpot-Pro** from [ss-v1](https://huggingface.co/datasets/rootsautomation/ScreenSpot) and [ss-pro](https://huggingface.co/datasets/likaixin/ScreenSpot-Pro), respectively.
For **ScreenSpot-v2**, we provide a converted version (`ScreenSpot-v2-new`) that aligns with the format used by the other datasets. However, you still need to download the original images from [ss-v2](https://huggingface.co/datasets/OS-Copilot/ScreenSpot-v2).


Once everything is set up, run the following commands:

```bash
bash run_ss_v1.sh
bash run_ss_v2.sh
bash run_ss_pro.sh
```





