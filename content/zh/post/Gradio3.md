---
title: Gradio官方学习文档（三）-Blocks
date: 2024-05-29T13:58:28+08:00
tags:
  - Gradio
featured_image: /images/Gradio.png
description: 📌是引用；💭 是个人观点
summary: Gradio官方学习文档第三节，Blocks模块
---
# 一、块和事件监听器
## 块的结构：
📌
```python
import gradio as gr


def greet(name):
    return "Hello " + name + "!"

#和interface不同的是，可以自定义label的名称，以及单个部件的构建
with gr.Blocks() as demo:
    name = gr.Textbox(label="Name")
    output = gr.Textbox(label="Output Box")
    greet_btn = gr.Button("Greet")
    greet_btn.click(fn=greet, inputs=name, outputs=output, api_name="greet")

demo.launch()


```
也可以在采用一个装饰器来在内部定义函数：
```python
import gradio as gr


with gr.Blocks() as demo:
    name = gr.Textbox(label="Name")
    output = gr.Textbox(label="Output Box")
    greet_btn = gr.Button("Greet")
	#在这里定义了一个装饰器，只需要写入输入参数和输出参数即可
    @greet_btn.click(inputs=name, outputs=output)
    def greet(name):
        return "Hello " + name + "!"

demo.launch()
```
显示结果如下：
![](/images/blockuse.png)
![[blockuse.png]]

💭Blocks相比较Interface来说，一个很明显的不同就是，Blocks可以自定义button，而Inerface模块基本使用就含有clear按钮和submit两个按钮。
interface模块ui如下：
![](/images/Progress-bars.png)
![[myblog/Gitreceiver.github.io/static/images/Progress-bars.png]]
## 二、事件侦听器和交互性
### 1.交互性：
在上面的示例中，您会注意到您可以编辑Name Textbox ，但不能编辑Output Textbox 。这是因为任何充当事件侦听器输入的组件都是交互式的。但是，由于Output Textbox 仅充当输出，因此 Gradio 确定不应将其设置为交互式。您可以覆盖默认行为，并使用 boolean 关键字参数interactive直接配置组件的交互性。📌

`output = gr.Textbox(label="Output", interactive=True)`
![](/images/interact.png)
![[myblog/Gitreceiver.github.io/static/images/interact.png]]

💭在这里的interactive关键字在Textbox组件中，标识是否是可交互。（即用户可否编辑）

### 2.事件侦听器

💭和之前提到的Interface中的及时响应（具体看Gradio官方学习文档（一））是一样的结果，但用法不同，这里采用事件监听器这个名称，具体用法是在上述代码中`with gr.Blocks() as demo:`下删除`button`相关的代码，添加`inp.change(welcome, inp, out)`，必须是对输入做监听。
```python
with gr.Blocks() as demo:
    gr.Markdown(
    """
    # Hello World!
    Start typing below to see the output.
    """)
    inp = gr.Textbox(placeholder="What is your name?")
    out = gr.Textbox()
    inp.change(welcome, inp, out)
```
📌该函数不是通过单击触发，而是通过键入文本框来触发。这是由于事件侦听器造成的。不同的组件支持不同的事件侦听器。例如，组件支持事件侦听器，该侦听器在用户按下播放键时触发。请参阅[文档](http://gradio.app/docs#components)，了解每个组件的事件侦听器。

## 三、多个数据流
📌Blocks 应用不像 Interfaces 那样局限于单个数据流。请看下面的演示：
`
```python

import gradio as gr

`def increase(num):`
    `return num + 1`

`with gr.Blocks() as demo:`
    `a = gr.Number(label="a")`
    `b = gr.Number(label="b")`
    `atob = gr.Button("a > b")`
    `btoa = gr.Button("b > a")`
    `atob.click(increase, a, b)`
    `btoa.click(increase, b, a)`

`demo.launch()`

```

💭这里也就是说，一个组件并不仅仅是输入或者是输出，可以作为一个数据流的输入和另一个数据流的输出。

📌举例：下面是一个“多步骤”演示示例，其中一个模型（语音转文本模型）的输出被馈送到下一个模型（情绪分类器）中。
```python
from transformers import pipeline

import gradio as gr

asr = pipeline("automatic-speech-recognition", "facebook/wav2vec2-base-960h")
classifier = pipeline("text-classification")


def speech_to_text(speech):
    text = asr(speech)["text"]
    return text


def text_to_sentiment(text):
    return classifier(text)[0]["label"]


demo = gr.Blocks()

with demo:
    audio_file = gr.Audio(type="filepath")
    text = gr.Textbox()
    label = gr.Label()

    b1 = gr.Button("Recognize Speech")
    b2 = gr.Button("Classify Sentiment")

    b1.click(speech_to_text, inputs=audio_file, outputs=text)
    b2.click(text_to_sentiment, inputs=text, outputs=label)

demo.launch()
```

![](/images/emotionrecog.png)
![[myblog/Gitreceiver.github.io/static/images/emotionrecog.png]]

## 四、函数输入列表与字典
📌到目前为止，您看到的事件侦听器具有单个输入组件。如果希望让多个输入组件将数据传递给函数，则有两个选项可以说明函数如何接受输入组件值：
1. 作为参数列表，或者
2. 作为单个值字典，由组件键控
让我们看一个例子：

```python
import gradio as gr

with gr.Blocks() as demo:
    a = gr.Number(label="a")
    b = gr.Number(label="b")
    with gr.Row():#这里表示按列分开两个按钮
        add_btn = gr.Button("Add")
        sub_btn = gr.Button("Subtract")
    c = gr.Number(label="sum")

    def add(num1, num2):
        return num1 + num2
    add_btn.click(add, inputs=[a, b], outputs=c)

    def sub(data):
        return data[a] - data[b]
    sub_btn.click(sub, inputs={a, b}, outputs=c)


demo.launch()
```

结果：
![](/images/multi-stream.png)
![[myblog/Gitreceiver.github.io/static/images/multi-stream.png]]
📌这是您喜欢哪种语法的偏好问题！对于具有许多输入组件的函数，选项 2 可能更易于管理。

## 五、函数返回列表与字典
📌同样，可以返回多个输出组件的值，如下所示：

1. 值列表
2. 由组件键入的字典
例一：
```python
with gr.Blocks() as demo:
    food_box = gr.Number(value=10, label="Food Count")
    status_box = gr.Textbox()
    def eat(food):
        if food > 0:
            return food - 1, "full"
        else:
            return 0, "hungry"
    gr.Button("EAT").click(
        fn=eat,
        inputs=food_box,
        #这里返回列表
        outputs=[food_box, status_box]
    )
```

上面，每个 return 语句分别返回两个对应的值。`food_box`和`status_box`

💭每点击一次eat，两个box都会发生变化，因为都作为输出流

除了按顺序返回与每个输出组件对应的值列表外，还可以返回一个字典，其中键对应于输出组件，值作为新值。这也**允许您跳过更新某些输出组件**
例二：

```python
with gr.Blocks() as demo:
    food_box = gr.Number(value=10, label="Food Count")
    status_box = gr.Textbox()
    def eat(food):
        if food > 0:
	        #区别在这里，返回的是一个字典
            return {food_box: food - 1, status_box: "full"}
        else:
	        #这里不更新food_box了，也就是上面说的【允许跳过更新某些输出组件】
            return {status_box: "hungry"}
    gr.Button("EAT").click(
        fn=eat,
        inputs=food_box,
        outputs=[food_box, status_box]
    )
```

![](/images/returnlist.png)
![[myblog/Gitreceiver.github.io/static/images/returnlist.png]]


💭所以总结下来，使用字典返回的好处就是，可以跳过更新某些输出组件（只返回更新的输出组件）

📌对于和事件侦听器的结合：

- 当事件侦听器在返回时影响许多组件，或者有条件地影响输出时，字典返回非常有用。

- 请记住，对于字典返回，我们仍然需要在事件侦听器中指定可能的输出。

## 六、更新组件配置

📌事件侦听器函数的返回值通常是相应输出组件的更新值。有时我们也想更新组件的配置，例如视觉效果（比如文本框的长短）。在本例中，我们返回一个新的组件，设置我们想要更改的属性（文本框长度）。
代码如下：
```python
import gradio as gr


def change_textbox(choice):
    if choice == "short":
        return gr.Textbox(lines=2, visible=True) # visible参数控制是否可见，如果value，lines和visible等参数未指定，则这些所有参数都将使用其以前的值。
    elif choice == "long":
        return gr.Textbox(lines=8, visible=True, value="Lorem ipsum dolor sit amet")
    else:
        return gr.Textbox(visible=False)


with gr.Blocks() as demo:
    radio = gr.Radio(
        ["short", "long", "none"], label="What kind of essay would you like to write?"
    )
    text = gr.Textbox(lines=2, interactive=True, show_copy_button=True)
    radio.change(fn=change_textbox, inputs=radio, outputs=text)


demo.launch()
```

💭visible参数控制是否可见，如果value，lines和visible等参数未指定，则这些所有参数都将使用其以前的值。

结果：

![](/images/newgr.png)
![[myblog/Gitreceiver.github.io/static/images/newgr.png]]

## 七、例子（输入建议）
在第一期的Interface输入建议已经提到，在此不多做赘述直接上代码：
```python
import gradio as gr


def calculator(num1, operation, num2):
    if operation == "add":
        return num1 + num2
    elif operation == "subtract":
        return num1 - num2
    elif operation == "multiply":
        return num1 * num2
    elif operation == "divide":
        return num1 / num2


with gr.Blocks() as demo:
    with gr.Row():
        with gr.Column():
            num_1 = gr.Number(value=4)
            operation = gr.Radio(["add", "subtract", "multiply", "divide"])
            num_2 = gr.Number(value=0)
            submit_btn = gr.Button(value="Calculate")
        with gr.Column():
            result = gr.Number()

    submit_btn.click(
        calculator, inputs=[num_1, operation, num_2], outputs=[result], api_name=False
    )
    examples = gr.Examples(
        examples=[
            [5, "add", 3],
            [4, "divide", 2],
            [-4, "multiply", 2.5],
            [0, "subtract", 1.2],
        ],
        inputs=[num_1, operation, num_2],
    )

if __name__ == "__main__":
    demo.launch(show_api=False)
```

📌**注**： 在 Gradio 4.0 或更高版本中，单击示例时，不仅输入组件的值会更新为示例值，而且组件的配置也会恢复为构造组件时使用的属性。这可确保示例与组件兼容，即使其配置已更改。

## 八、连续运行事件（then）
📌例如，在下面的聊天机器人示例中，我们首先立即使用用户消息更新聊天机器人，然后在模拟延迟后使用计算机响应更新聊天机器人。
代码：
```python
import gradio as gr
import random
import time

with gr.Blocks() as demo:
    chatbot = gr.Chatbot()
    msg = gr.Textbox()
    clear = gr.Button("Clear")

    def user(user_message, history):
        return "", history + [[user_message, None]]

    def bot(history):
        bot_message = random.choice(["How are you?", "I love you", "I'm very hungry"])
        time.sleep(2)
        history[-1][1] = bot_message
        return history

    msg.submit(user, [msg, chatbot], [msg, chatbot], queue=False).then(
        bot, chatbot, chatbot
    )
    clear.click(lambda: None, None, chatbot, queue=False)
    
demo.queue()
demo.launch()
```

📌注：
事件侦听器的方法执行后续事件，而不管前一个事件是否引发任何错误。如果只想在前一个事件成功执行时运行后续事件，请使用该方法，该方法采用与 相同的参数。`.then()``.success()``.then()`


接下来的就有难度了，连续运行进阶：
📌可以使用事件侦听器的参数按固定计划运行事件。这将在客户端连接打开时运行事件秒数。如果连接关闭，则该事件将在下一次迭代后停止运行。请注意，这未考虑事件本身的运行时。因此，运行时间为 1 秒的函数实际上每 6 秒运行一次。另请注意，此参数不适用于函数，仅适用于与事件侦听器关联的 Python 函数。（是不是看不太懂，我也是）

```python
import math
import gradio as gr
import plotly.express as px
import numpy as np


plot_end = 2 * math.pi


def get_plot(period=1):
    global plot_end
    x = np.arange(plot_end - 2 * math.pi, plot_end, 0.02)
    y = np.sin(2*math.pi*period * x)
    fig = px.line(x=x, y=y)
    plot_end += 2 * math.pi
    if plot_end > 1000:
        plot_end = 2 * math.pi
    return fig


with gr.Blocks() as demo:
    with gr.Row():
        with gr.Column():
            gr.Markdown("Change the value of the slider to automatically update the plot")
            period = gr.Slider(label="Period of plot", value=1, minimum=0, maximum=10, step=1)
            plot = gr.Plot(label="Plot (updates every half second)")
	
	#把下面这一行和cancels=[dep]删除了也能跑
    dep = demo.load(get_plot, None, plot, every=1)
    period.change(get_plot, period, plot, every=1, cancels=[dep])


if __name__ == "__main__":
    demo.queue().launch()
```

我运行了他的代码，结果是拖到高频，再拉到低频看，好嘛，已经被拉废了：

![](/images/sinx.gif)
![[myblog/Gitreceiver.github.io/static/images/sinx.gif]]

所以官方代码也还有改进空间。

💭在这里更想提一些代码的细节问题，也就是代码的逻辑。
首先看一下`get_plot()`函数的返回值，`return fig`，而`fig=px.line`，在`period.change(get_plot, period, plot, every=1, cancels=[dep])`中对应的返回值为`gr.Plot`，也就是说，`fig=px.line`和`gr.Plot`在Gradio中是耦合的。
其次，看一下main函数上的最后两行代码，这两行代码的实现逻辑没有弄懂，并且将倒数第二行代码和最后一行代码中的`cancels=[dep]`删除，代码照样可以运行，也还是有函数重合闪现的问题。（如果有懂得可以摇我）
而实现监控只需要`period.change(get_plot, period, plot, every=1)`这一行代码已经可以了，为什么要用句柄dep。


## 九、收集事件数据
📌方法：**通过将关联的事件数据类作为类型提示添加到事件侦听器函数中的参数中**，可以收集有关事件的特定数据。
2 人井字游戏演示中，用户可以在其中选择一个单元格进行移动。event data 参数包含有关所选特定单元格的信息。我们可以先检查单元格是否为空，然后使用用户的移动更新单元格。
结果如下：
![](/images/XO.gif)
![[XO.gif]]
代码：
```python
import gradio as gr

with gr.Blocks() as demo:
    turn = gr.Textbox("X", interactive=False, label="Turn")
    board = gr.Dataframe(value=[["", "", ""]] * 3, interactive=False, type="array")

    def place(board, turn, evt: gr.SelectData):
	    #检查是否已经被点击，被点击过（有值），那么保持不变
        if evt.value:
            return board, turn
        #获取点击事件所选择数据的行和列，置为turn
        board[evt.index[0]][evt.index[1]] = turn
        turn = "O" if turn == "X" else "X" #反置
        return board, turn
	#注意输入不包含evt
    board.select(place, [board, turn], [board, turn], show_progress="hidden")

demo.launch()
```
💭在这里，代码中没有上文提到的事件监听器`change()`方法的字眼，却实现了事件监听功能，因为这里多了一个函数参数`evt: gr.SelectData`。直接使用**事件数据类**`evt: gr.SelectData`即可实现监听器的功能。
注意一点就是，在调用的时候，`board.select(place, [board, turn], [board, turn], show_progress="hidden")`输入为`[board, turn]`，并**没有**`evt`这个参数输入。


## 十、将多个触发器绑定到一个函数
💭所谓的“牵一发而动全身”。（一个开关控制多个线程）
代码：
```python
import gradio as gr

with gr.Blocks() as demo:
    name = gr.Textbox(label="Name")
    output = gr.Textbox(label="Output Box")
    greet_btn = gr.Button("Greet")
    trigger = gr.Textbox(label="Trigger Box")
    trigger2 = gr.Textbox(label="Trigger Box")

    def greet(name, evt_data: gr.EventData):
        return "Hello " + name + "!", evt_data.target.__class__.__name__
    
    def clear_name(evt_data: gr.EventData):
        return "", evt_data.target.__class__.__name__
    
    gr.on(
        triggers=[name.submit, greet_btn.click],
        fn=greet,
        inputs=name,
        outputs=[output, trigger],
    ).then(clear_name, outputs=[name, trigger2])


demo.launch()
```
结果：
![](/images/multitri.gif)
![[myblog/Gitreceiver.github.io/static/images/multitri.gif]]也可以采用装饰器gr.on()：
```python
import gradio as gr

with gr.Blocks() as demo:
    name = gr.Textbox(label="Name")
    output = gr.Textbox(label="Output Box")
    greet_btn = gr.Button("Greet")

    @gr.on(triggers=[name.submit, greet_btn.click], inputs=name, outputs=output)
    def greet(name):
        return "Hello " + name + "!"


demo.launch()
```
在上面的代码中，装饰器 `@gr.on`中三个参数 `triggers，inputs，outputs`，如果不指定triggers，那么input里面任何的元素发生改变都会触发事件（函数`greet`运行），如果指定，那么只有当制定的事件发生，函数才会执行。比如上面代码制定了`triggers=[name.submit, greet_btn.click]`，那么只有在`name`栏回车（可不输入内容），或者点击`Greet`按钮才会触发事件。

不指定`triggers`的例子
代码：
```python
import gradio as gr

with gr.Blocks() as demo:
    with gr.Row():
        num1 = gr.Slider(1, 10)
        num2 = gr.Slider(1, 10)
        num3 = gr.Slider(1, 10)
    output = gr.Number(label="Sum")

    @gr.on(inputs=[num1, num2, num3], outputs=output)
    def sum(a, b, c):
        return a + b + c
```
![](/images/gron-notrig.gif)
![[myblog/Gitreceiver.github.io/static/images/gron-notrig.gif]]