---
title: Gradio官方学习文档（一）
date: 2024-05-22T15:27:15+08:00
tags:
  - Gradio
featured_image: /images/Gradio.png
description: 📌是引用；💭 是个人观点
summary: Gradio官方学习文档第一节
---
## 基本使用方式：


```python

import gradio as gr

#参数名直接就是界面显示的参数名，无需再次定义界面head名称
def greet(name, intensity):
    return "Hello, " + name + "!" * int(intensity)

demo = gr.Interface(
    fn=greet, #按钮submit链接的函数
    inputs=["text", "slider"], # 函数输入类型text为文字框，slider为拖拉框（需要和函数定义吻合）
    outputs=["text"], # 函数输出，也需要和函数输出吻合
    
    #以下是非必需
	title="Toy Calculator", #标题
    description="Here's a sample toy calculator. Allows you to calculate things like $2+2=4$" #说明
)

# 也可以采用这个方式
demo = gr.Interface(
    fn=greet,
    inputs=["text", gr.Slider(value=2, minimum=1, maximum=10, step=1)],
    outputs=[gr.Textbox(label="greeting", lines=3)] #将“text”改为一个组件gr.Textbox的格式,自定义label和长度。
)
#而对于单个输入输出，简化可以为：
demo = gr.Interface(sepia, gr.Image(), "image")#输入图像（默认为array格式）和输出图像

#如果想让输入图片为‘路径格式’，只需要将gr.Image()修改为
gr.Image(type="filepath", shape=...)

demo.launch()
```

## gr.Radio选择框定义与实现与examples
```python
import gradio as gr
#from foo import BAR
#
def calculator(num1, operation, num2):
    if operation == "add":
        return num1 + num2
    elif operation == "subtract":
        return num1 - num2
    elif operation == "multiply":
        return num1 * num2
    elif operation == "divide":
        if num2 == 0:
            raise gr.Error("Cannot divide by zero!")
        return num1 / num2

demo = gr.Interface(
#另外一种调用格式，直接写出函数、输入、输出
    calculator,
    [
        "number", 
        gr.Radio(["add", "subtract", "multiply", "divide"]),
        "number"
    ],
    "number",
    
    #非必选，在下方给出示例
    examples=[
        [45, "add", 3],
        [3.14, "divide", 2],
        [144, "multiply", 2.5],
        [0, "subtract", 1.2],
    ],
    title="Toy Calculator",
    description="Here's a sample toy calculator. Allows you to calculate things like $2+2=4$",
)

demo.launch()
```
添加了examples过后：
![](/images/gradio示例.png)
![[gradio示例.png]]
## flag解析
输出结果CSV格式文件到flagged文件夹中
比如上面的四则运算结果：
+-- calculator.py

+-- flagged/

|   +-- logs.csv

logs.csv内容为：
num1,operation,num2,Output

5,add,7,12

6,subtract,1.5,4.5

或者上文图片的输入输出结果：
+-- sepia.py

+-- flagged/

|   +-- logs.csv

|   +-- im/

|   |   +-- 0.png

|   |   +-- 1.png

|   +-- Output/

|   |   +-- 0.png

|   |   +-- 1.png

其中的_flagged/logs.csv文件内容是：

im,Output

im/0.png,Output/0.png

im/1.png,Output/1.png

## interface state（具体实例为历史记录的保存）
方法一：全局变量（略过，因为和python中的全局变量使用方法一致）
方法二： Session State
和全局变量的不同之处在于，重新刷新页面以及切换user，states不再保存之前的信息。
```python
import gradio as gr

def store_message(message: str, history: list[str]):
    output = {
        "Current messages": message,
        "Previous messages": history[::-1]
    }
    history.append(message)
    return output, history

#其中inputs里的gr.state(value[])初始化一个存储历史记录的数组，output中的gr.state()用来表示一个可以随着全局变化的变量
demo = gr.Interface(fn=store_message, 
                    inputs=["textbox", gr.State(value=[])], 
                    outputs=["json", gr.State()])

demo.launch()
```
![](/images/interface-state.png)
![[interface-state.png]]
注：接口类仅支持单个会话状态变量（尽管它可以是一个包含多个元素的列表）。对于更复杂的使用场景，您可以使用块（Blocks），它支持多个状态变量。或者，如果您正在构建一个维护用户状态的聊天机器人，可以考虑使用ChatInterface抽象，它会自动管理状态。

## 及时响应和连续流式数据
只需要在gr.Interface()中添加live=True即可，添加后没有了submit按钮，即使响应结果.
```python
demo = gr.Interface(
    calculator,
    [
        "number",
        gr.Radio(["add", "subtract", "multiply", "divide"]),
        "number"
    ],
    "number",
    live=True,#添加这一行
)
```

![](/images/Live-Interfaces.png)
![[Live-Interfaces.png]]

但需要考虑其他问题，比如四则运算运行时会出现错误问题。
![](/images/Live-Interfaces1.png)
![[Live-Interfaces1.png]]


一些组件拥有“流式”模式，比如以麦克风模式运行的音频组件，或以网络摄像头模式运行的图像组件。
`gr.Audio(source='microphone')` and `gr.Audio(source='microphone', streaming=True)`这两者之间的区别：
第一个组件将在用户停止录制时自动提交数据并运行接口函数（一次转发），而第二个组件将在录制过程中持续发送数据并运行接口函数。（流式）

输出组件可应用在输入流和输出流。（双向）

示例：
```python
import gradio as gr
from pydub import AudioSegment
from time import sleep

with gr.Blocks() as demo:
    input_audio = gr.Audio(label="Input Audio", type="filepath", format="mp3")
    with gr.Row():
        with gr.Column():
            stream_as_file_btn = gr.Button("Stream as File")
            format = gr.Radio(["wav", "mp3"], value="wav", label="Format")
            stream_as_file_output = gr.Audio(streaming=True)

            def stream_file(audio_file, format):
                audio = AudioSegment.from_file(audio_file)
                i = 0
                chunk_size = 1000
                while chunk_size * i < len(audio):
                    chunk = audio[chunk_size * i : chunk_size * (i + 1)]
                    i += 1
                    if chunk:
                        file = f"/tmp/{i}.{format}"
                        chunk.export(file, format=format)
                        yield file
                        sleep(0.5)

            stream_as_file_btn.click(
                stream_file, [input_audio, format], stream_as_file_output
            )

            gr.Examples(
                [["audio/cantina.wav", "wav"], ["audio/cantina.wav", "mp3"]],
                [input_audio, format],
                fn=stream_file,
                outputs=stream_as_file_output,
            )

        with gr.Column():
            stream_as_bytes_btn = gr.Button("Stream as Bytes")
            stream_as_bytes_output = gr.Audio(format="bytes", streaming=True)

            def stream_bytes(audio_file):
                chunk_size = 20_000
                with open(audio_file, "rb") as f:
                    while True:
                        chunk = f.read(chunk_size)
                        if chunk:
                            yield chunk
                            sleep(1)
                        else:
                            break
            stream_as_bytes_btn.click(stream_bytes, input_audio, stream_as_bytes_output)

if __name__ == "__main__":
    demo.queue().launch()

```

## 四种gradio interface
1. 标准演示：既有独立的输入又有输出（例如图像分类器或语音转文本模型） 


![](/images/标准.png)
![[标准.png]]
2. 仅输出演示：不接受任何输入但会产生输出（例如无条件图像生成模型）
仅输入：（也就是无函数参数，以及将gr.Interface中的inputs置为None）
```python
def fake_gan():
    time.sleep(1)
    images = [
            "https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=387&q=80",
            "https://images.unsplash.com/photo-1554151228-14d9def656e4?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=386&q=80",
            "https://images.unsplash.com/photo-1542909168-82c3e7fdca5c?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxzZWFyY2h8MXx8aHVtYW4lMjBmYWNlfGVufDB8fDB8fA%3D%3D&w=1000&q=80",
    ]
    return images


demo = gr.Interface(
    fn=fake_gan,
    inputs=None,
    outputs=gr.Gallery(label="Generated Images", columns=[2]),
    title="FD-GAN",
    description="This is a fake demo of a GAN. In reality, the images are randomly chosen from Unsplash.",
)
```
3. 仅输入演示：不产生任何输出但能接受某种输入（例如将您上传的图像保存到持久性外部数据库的演示） 
仅输出：（同理是将函数返回值取消，以及将gr.Interface中的outputs置为None）
```python
def save_image_random_name(image):
    random_string = ''.join(random.choices(string.ascii_letters, k=20)) + '.png'
    image.save(random_string)
    print(f"Saved image to {random_string}!")

demo = gr.Interface(
    fn=save_image_random_name, 
    inputs=gr.Image(type="pil"), 
    outputs=None,
)
```

4. 统一演示：既有输入组件又有输出组件，但**输入和输出组件相同**。这意味着生成的输出会覆盖输入（例如文本自动完成模型）

```python
import gradio as gr
from transformers import pipeline

generator = pipeline('text-generation', model = 'gpt2')

def generate_text(text_prompt):
  response = generator(text_prompt, max_length = 30, num_return_sequences=5)
  return response[0]['generated_text']

textbox = gr.Textbox()

demo = gr.Interface(generate_text, textbox, textbox)

demo.launch()

```
