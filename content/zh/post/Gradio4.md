---
title: Gradio官方学习文档（四）-Blocks控制布局
date: 2024-05-30T13:58:28+08:00
tags:
  - Gradio
featured_image: /images/Gradio.png
description: 📌是引用；💭 是个人观点
summary: Gradio官方学习文档第四节，Blocks模块构建后续
---
1.多个菜单栏：**with gr.Tab()**

2.图片编辑器：**ImageEditor**
```python
import gradio as gr

with gr.Blocks() as demo:
    im = gr.ImageEditor(
        width="50vw",
    )

demo.launch()
```
vm(px)代表组件所占blocks的比例大小

3.同样控制部件所占比例大小的方法还有**height=**：
```python
with gr.Column(elem_classes=["container"]):
        name = gr.Chatbot(value=[["1", "2"]], height="70%")
```

4.下拉扩展栏**with gr.Accordion**
```python
with gr.Accordion("Open for More!", open=False):
```

5.文件上传模块：**gr.File(label="上传文件")**
