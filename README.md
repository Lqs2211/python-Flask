# python-Flask
python
from flask import Flask, jsonify, render_template
import requests
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

# 首先获取访问令牌
def get_token():
    url = "http://api.nlecloud.com/Users/Login"
    # 准备登录的参数
    p = {
        "Account": "19000000011",
        "Password": "000000",
        "IsRememberMe": True
    }
    # 采用JSON格式封装用户身份信息
    r = requests.post(url, json=p)
    d = r.json()
    # 解析响应数据并提取访问令牌
    return d["ResultObj"]["AccessToken"]

# 接下来获取单个传感器数据
def get_data(d_id, api_tag, token):
    # 将设备id和传感器类型作为参数
    url = f"http://api.nlecloud.com/devices/{d_id}/Sensors/{api_tag}"
    h = {"AccessToken": token}
    # 在请求头中注入认证令牌
    r = requests.get(url, headers=h)
    d = r.json()
    # 返回标准化数据
    return d["ResultObj"]

# 接下来获取所有传感器数据
def get_all_sensors_data(d_id, token):
    # 定义传感器标签与名称的映射字典
    sensors = {
        "guangzhao": "光照传感器",
        "wendu": "温度传感器",
        "shidu": "湿度传感器"
    }

    # 接下来初始化一个空字典
    y = {}
    # 遍历传感器配置
    for sensor_tag, sensor_name in sensors.items():
        # 调用函数获取数据
        data = get_data(d_id, sensor_tag, token)
        # 将数据存储到结果字典中
        y[sensor_tag] = {
            "name": sensor_name,
            "data": data
        }

    return y  # 返回结果字典

# 接下来定义装饰器
@app.route('/')
def index():
    # 渲染并返回index.html（因戴可思 html）模板
    return render_template('index.html')

# 接下来定义API路由
@app.route('/api/sensor-data')
def get_sensor_data():
    # 获取访问令牌
    token = get_token()
    # 设置设备 ID
    device_id = 1368228
    # 获取所有传感器数据
    all_sensors_data = get_all_sensors_data(device_id, token)
    # 将数据转换为 JSON 格式返回
    return jsonify(all_sensors_data)

# 接下来判断是否直接运行此脚本
if __name__ == '__main__':
    # 启用调试模式,设置端口为5000
    app.run(debug=True, host='0.0.0.0', port=5000)
