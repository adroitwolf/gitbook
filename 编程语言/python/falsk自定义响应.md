# flask 自定义响应

## 返回图片

正常情况下，我们不需要返回图片或者其他格式的流文件，只需要把存放在本地的图片地址发送给前端即可。这里仅仅是提供发送文件的一种思路。
使用flask的send_file()函数，具体使用方法如下

```python
@app.route('/func1',methods=['POST'])
def hello_world():
    pic = request.files['file']
    img = pic.read()
    return send_file(io.BytesIO(img), mimetype='image/jpeg')

```

## 返回图片和文本格式混合的自定义格式

这里需要将图片等文件先转成base64编码，然后提供给前端，最后前端再从base64转成图片即可。

```python


@app.route('/func1',methods=['POST'])
def func1():
    pic = request.files['file']
    
    img = pic.read()
    # 这里就是正常的img，之前怎么弄得，这里就怎么弄
    img = base64.encodebytes(io.BytesIO(img).getvalue()).decode('ascii')
    
    # 返回一个图片的base64格式，必须是png格式，如果不是，format一下
    res = {}
    res['file'] = img
    # 这边的数据模型给出来
    res['data'] = ['car','person','bench']
    resp = make_response(res)
    return resp
```