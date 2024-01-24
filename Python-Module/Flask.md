### 기본적인 사용법
- index.html
```html
<html>	
	<head>
	</head>
	
	<body>	
	{% if True %}
		<img src="{{ url_for('static', filename=image_file) }}">
	{% endif %}
	</body>
</html>
```
- viewer.py
```python
import random
from flask import Flask, render_template
app = Flask(__name__)

@app.route("/")
def home():
	dynamic_image_file = get_dynamic_image()
	return render_template('index.html', image_file=dynamic_image_file)
	
def get_dynamic_image():
# 50%의 확률로 첫 번째 이미지 또는 두 번째 이미지를 선택
	dynamic_image = random.choice(["image/test.png", "image/test2.png"])
	return dynamic_image

if __name__ == "__main__":
	app.run(debug=True, port=6006)
```
- Flask의 기본 폴더 구조
	- [Flask_root/templates] 에 html 형식의 template를 저장
	- [Flask_root/static] 에 정적인 파일을 저장
	- `url_for` 함수를 활용해 파일 경로를 맞춰준다


### 이미지 동적 변경 (Socketio)
- index.html
```html
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Image Viewer with SocketIO</title>
</head>
<body>
	<img id="image" alt="Image">
	<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/3.0.4/socket.io.js"></script>
<script>
	var socket = io.connect('http://' + document.domain + ':' + location.port);
	// 페이지 로드 시 처음 한 번 이미지를 요청하고 설정
	requestNewImage();
	
	// 일정 간격으로 이미지를 요청합니다.
	setInterval(requestNewImage, 500);
	
	// 이미지 요청 함수
	function requestNewImage() {
		socket.emit('request_new_image');
	}
	// 새 이미지가 도착하면 이미지를 갱신합니다.
	socket.on('new_image', function(data) {
		document.getElementById('image').src = '/static/' + data.image_file;
	});
</script>
</body>
</html>
```
- viewer.py
```python
from flask import Flask, render_template
from flask_socketio import SocketIO
import random

app = Flask(__name__)
socketio = SocketIO(app)

@app.route("/")
def home():
	return render_template('index.html')

@socketio.on('request_new_image')
def handle_request_new_image():
	dynamic_image_file = get_dynamic_image()
	socketio.emit('new_image', {'image_file': dynamic_image_file})

def get_dynamic_image():
	# 50%의 확률로 첫 번째 이미지 또는 두 번째 이미지를 선택
	dynamic_image = random.choice(["image/test.png", "image/test2.png"])
	return dynamic_image

if __name__ == "__main__":
	socketio.run(app, debug=True, port=6006)
```
*get_dynamic_image() 내부의 조건이나 반복문을 변경하여 원하는대로 이미지 변경*
