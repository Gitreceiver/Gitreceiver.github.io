---
title: Gradio官方学习文档（二）
date: 2024-05-25T09:41:00+08:00
tags:
  - Gradio
featured_image: /images/Gradio.png
description: 📌是引用；💭 是个人观点
summary: Gradio官方学习文档第二节
---
## 一、队列

多用户访问Gradio app排队机制，支持的类：`gr.Interface`, `gr.Blocks`, and `gr.ChatInterface`
示例：
```python
#在类后使用queue()函数，设定`default_concurrency_limit` = 5（不声明，默认值为1）
demo = gr.Interface(...).queue(default_concurrency_limit=5)
demo.launch()
```

## 二、流式输出（yield）
在Gradio中，streaming=True 和生成器 yield 都是用于处理实时数据流的技术，但它们在工作方式和应用场景上有所不同。

### `streaming=True`
`streaming=True`目前只支持Audio和Image类
- 使用 `streaming=True` 可以将数据流拆分成多个小块，并在每个小块准备好后立即将其发送给Gradio界面。这使得Gradio能够实时更新界面，而无需等待整个数据流完全准备好。
- 使用 `streaming=True` 需要使用 `yield` 生成器来逐个生成数据块。
- `streaming=True` 适用于需要**实时更新界面**的场景，例如实时监控传感器数据或生成实时音频/视频。
示例：`gr.Audio(source='microphone', streaming=True)`

### **生成器 `yield`**

- 生成器 `yield` 是一种用于生成序列的函数。它每次调用时都会返回序列中的下一个元素，但不会立即计算整个序列。
- 生成器 `yield` 可以与 `streaming=True` 一起使用，也可以单独使用。
- 生成器 `yield` 适用于需要**按需生成数据**的场景，例如生成大型数据集或进行**迭代**计算。

区别对比：
|   |   |   |
|---|---|---|
|特性|streaming=True|生成器 yield|
|工作方式|将数据流拆分成多个小块，并在每个小块准备好后立即将其发送给Gradio界面。|每次调用时都会返回序列中的下一个元素，但不会立即计算整个序列。|
|应用场景|**实时更新界面**|按需**生成数据**|
|适用范围|Audio和Image类|几乎所有|

示例：
```python

import gradio as gr
import numpy as np
import time
#Steps步渲染出目标图片，实现流式输出
def fake_diffusion(steps):
    rng = np.random.default_rng()
    for i in range(steps):
        time.sleep(1)
        image = rng.random(size=(600, 600, 3))
        yield image
    image = np.ones((1000,1000,3), np.uint8)
    image[:] = [255, 124, 0]
    yield image


demo = gr.Interface(fake_diffusion,
                    inputs=gr.Slider(1, 10, 3, step=1),#定义拖拉条，默认值为3，拖拉单位为1
                    outputs="image")

demo.launch()

```

![](/images/yield1.png)
![[yield1.png]]
## 三、警示通知
报错：

    gr.Error('This is an error.')    

通知：

    gr.Info('This is some info.')

    gr.Warning('This is a warning.')
    



   Info和Warning这两个区别只是颜色区别
   ![](/images/alert.png)

![[alert.png]]

## 四、主题风格
使用方法
```python
demo = gr.Interface(..., theme=gr.themes.Monochrome())
```

要进一步拥有样式设计的能力，您可以向Gradio应用程序传递任何CSS（以及自定义JavaScript）。

预设主题切换示例：
```python
import gradio as gr

gr.themes.builder()
```
![](/images/theme-mon.png)
![[theme-mon.png]]
![](/images/theme-soft.png)
![[theme-soft.png]]

其他主题的细节设置请看官访文档：

https://www.gradio.app/guides/theming-guide

## 五、进度条

示例：
```python
import gradio as gr
import time

#在函数定义添加参数gr.Progress()即可，inputs和outputs不需要改变
def slowly_reverse(word, progress=gr.Progress()):
    #初始化
    progress(0, desc="Starting")
    time.sleep(1)
    #定义处理的显示单位长度
    progress(0.05)
    new_string = ""
    for letter in progress.tqdm(word, desc="Reversing"):
        time.sleep(0.25)
        new_string = letter + new_string
    return new_string

demo = gr.Interface(slowly_reverse, gr.Text(), gr.Text())

demo.launch()

```
也可以和tqdm进行耦合，如果想使用tqdm，只需要修改为gr.Progress(track_tqdm=True)，前提是有tqdm.tqdm在你的函数体内。
![](/images/Progress-bars.png)
![[Progress bars.png]]

## 六、批处理函数（对多个请求，并行处理加速）
Gradio支持传递批处理函数的功能。批处理函数就是接收输入列表并返回预测列表的函数。
例如，这里是一个批处理函数，它接收两个输入列表（一个单词列表和一个整数列表），并返回修剪后的单词列表作为输出：
```python
import time

def trim_words(words, lens):
    trimmed_words = []
    time.sleep(5)
    for w, l in zip(words, lens):
        trimmed_words.append(w[:int(l)])
    return [trimmed_words]
```
使用批处理函数的优势在于，如果启用排队，Gradio服务器可以自动批处理传入的请求并并行处理，可能加快演示速度。
以下是Gradio代码示例（请注意batch=True和max_batch_size=16）：
gr.Interface类使用方式：
```python
demo = gr.Interface(
    fn=trim_words, 
    inputs=["textbox", "number"], 
    outputs=["output"],
    batch=True, 
    max_batch_size=16
)

demo.launch()
```
gr.Blocks使用方式（之后会提到gr.Blocks）
```python
import gradio as gr

with gr.Blocks() as demo:
    with gr.Row():
        word = gr.Textbox(label="word")
        leng = gr.Number(label="leng")
        output = gr.Textbox(label="Output")
    with gr.Row():
        run = gr.Button()

    event = run.click(trim_words, [word, leng], output, batch=True, max_batch_size=16)

demo.launch()
```

在上面的例子中，可以并行处理16个请求（总推理时间为5秒），而不是每个请求单独处理（总推理时间为80秒）。许多 Hugging Face 的变换器和扩散模型在 Gradio 的批处理模式下工作得非常自然。