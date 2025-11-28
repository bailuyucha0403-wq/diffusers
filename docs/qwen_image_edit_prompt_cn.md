### Qwen Image Edit 对提示的理解方式

- Qwen-Image-Edit 将 **Qwen2.5-VL-7B-Instruct** 多模态大模型作为文本编码器，并使用 `Qwen2VLProcessor`，因此对提示的理解委托给能够同时处理文本与视觉输入的 VL 模型。

- 流水线会把用户指令和输入图像封装进聊天式模板（`<|im_start|>system ... <|vision_start|><|image_pad|><|vision_end|>{prompt}`），保留图像 token；固定的 `prompt_template_encode_start_idx` 控制稍后要丢弃的前缀 token。

- `_get_qwen_prompt_embeds` 通过处理器构建多模态张量（文本加图像），用 `Qwen2_5_VLForConditionalGeneration` 在 `output_hidden_states=True` 下处理，然后从最后一层隐藏状态中舍弃前 64 个 token，再将得到的嵌入和注意力掩码传给扩散 Transformer。这意味着 Diffusion 模型使用的是 VL 模型的最终隐藏状态（减去提示前缀）来“理解”编辑指令。
