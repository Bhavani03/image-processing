# image-processing
import numpy as np
from scipy.misc import imresize, imread, imsave
import tensorflow as tf
from keras.models import load_model

model = load_model('./utils/unet.hdf5')
graph = tf.get_default_graph()

def get_colored(gray_path='./static/gray.jpg', color_path='./static/color.jpg'):
    gray = imresize(imread(gray_path), (32,32))
    if gray.shape[2]>=3: gray = gray.sum(axis=-1)/3
    gray = gray/255
    with graph.as_default():
        color = model.predict(gray.reshape((1,32,32,1)))[0]
    imsave(color_path, imresize(color, (200,200)))
    imsave(gray_path, imresize(gray, (200,200)))
 
 
 
 
 
 
    #---FLASK APP---
from flask import Flask, render_template, request
from colorize import get_colored
import os
app = Flask(__name__)

@app.route('/', methods = ['GET', 'POST'])
def upload_file():
    gray_path = './static/gray.jpg'
    color_path = './static/color.jpg'
    if os.path.isfile(gray_path): os.remove(gray_path)
    if os.path.isfile(color_path): os.remove(color_path)

    if request.method == 'POST':
        f = request.files.get('file')
        if f is None:
        	return "No Image Uploaded!!!"
        f.save(gray_path)
        get_colored()
        return render_template("index.html", after=True)
    return render_template("index.html", after=False)

@app.after_request
def add_header(response):
    response.headers['X-UA-Compatible'] = 'IE=Edge,chrome=1,firefox=1'
    response.headers['Cache-Control'] = "no-cache, no-store, must-revalidate"
    response.headers["Pragma"] = "no-cache"
    response.headers["Expires"] = "0"
    response.headers['Cache-Control'] = 'public, max-age=0'
    return response
    
    
    
    
    
    
    
    
    <html>
<head>
  <title>Image Colorizer</title>
</head>
<body>
  <center><h1>Image Colorizer</h1>
  <form action = "" method = "POST" enctype = "multipart/form-data">
    <input type = "file" name = "file">
    <input type = "submit" value="Upload Image">
  </form></center>
  {% if after %}
  <br/><center>
    <img src="{{url_for('static', filename='gray.jpg')}}" width="200" style="margin:6vw;border:2px solid black;">
    <img src="{{url_for('static', filename='color.jpg')}}" width="200" style="margin:6vw;border:2px solid black;">
  </center>
  {% endif %}
</body>
</html>





