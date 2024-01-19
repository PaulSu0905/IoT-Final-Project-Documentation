# IoT Final Project Documentation

###### things you need

* pixy2 CMUcam5 https://www.taiwaniot.com.tw/product/pixy2-cmucam5-影像圖形辨識系統/
* Pixy2 Pan-tilt – Dual Axis Robotic Camera  Mount https://www.taiwaniot.com.tw/product/grove-pixy2-dual-axis-robotic-camera-mount/
* Raspberry PI RPI CAMERA(若能使用hd版更好) https://www.taiwaniot.com.tw/product/樹莓派專用攝影鏡頭模組-raspberry-pi-rpi-camera-500萬像素/
* NCS https://www.intel.com/content/www/us/en/developer/articles/tool/neural-compute-stick.html

## Geting started


### Step 1: 組裝pixy雲台
根據以下組裝文件完成雲台組裝（hint:建議使用自備品質較好之usb線，供電及連接會較穩定）

雲台組裝文件：https://docs.pixycam.com/wiki/doku.php?id=wiki:v2:assembling_pantilt_mechanism
![img_5226_result](https://hackmd.io/_uploads/BkepPKYt_p.jpg)


### Step 2: 在樹莓派上安裝 pixymon
請根據以下安裝教學安裝pixymon至樹莓派

pixymon安裝教學：https://docs.pixycam.com/wiki/doku.php?id=wiki:v2:installing_pixymon_on_linux

如在sudo apt-get install qt5-default此步驟出現問題可嘗試
```
# Install software-properties-common
sudo apt-get update
sudo apt-get install software-properties-common

# Add the Qt5 repository
sudo add-apt-repository ppa:qt5-xenial/qt5

# Update package lists
sudo apt-get update

# Install qt5-default
sudo apt-get install qt5-default

```
之後開啟pixymon可透過以下指令

```
cd pixy2/build/pixymon/
./PixyMon
```


### Step 3: 教pixy辨識一個你想追蹤的物品

請根據以下辨識教學教會pixy你所想追蹤拍攝的物品

pixy辨識物品教學： https://docs.pixycam.com/wiki/doku.php?id=wiki:v2:teach_pixy_an_object


### Step 4: 測試雲台是否能夠正常運作
請根據以下Pan-tilt Demo文件測試雲台是否能正確追蹤物體

Pan-tilt Demo：https://docs.pixycam.com/wiki/doku.php?id=wiki:v2:run_the_pantilt_demo


### Step 5測試picamera:
請根據以下文件測試picamera是否能正常使用，如遇到裝上picamera後開啟樹莓派黑頻之情況請查看下方補充。

picamera文件：https://picamera.readthedocs.io/en/release-1.13/
**補充** 黑頻處理（參考情況三）：https://blog.csdn.net/qq_43619832/article/details/124243048
### Step 6撰寫picamera程式:
建立一個資料夾當中須包含
* CameraApp.py
* photos(資料夾)
* videos（資料夾）
以下為CameraApp.py中的程式碼
```
import tkinter as tk
from picamera import PiCamera
from time import sleep
from PIL import Image, ImageTk
import os
import io

class CameraApp:
    def __init__(self, master):
        self.master = master
        self.master.title("PiCamera Controller")

        self.camera = PiCamera()
        self.camera.resolution = (640, 480)

        # Label to display video stream
        self.video_label = tk.Label(self.master)
        self.video_label.pack()

        # Label to display messages
        self.message_label = tk.Label(self.master, text="")
        self.message_label.pack()

        # Buttons for capturing photo and starting/stopping recording
        self.capture_button = tk.Button(self.master, text="拍照", command=self.capture_photo)
        self.capture_button.pack(side=tk.LEFT, padx=10, pady=5)

        self.record_button = tk.Button(self.master, text="開始錄影", command=self.start_recording)
        self.record_button.pack(side=tk.LEFT, padx=10, pady=5)

        self.stop_record_button = tk.Button(self.master, text="停止錄影", command=self.stop_recording, state=tk.DISABLED)
        self.stop_record_button.pack(side=tk.LEFT, padx=10, pady=5)

        # Counter for naming photos and videos
        self.photo_counter = 1
        self.video_counter = 1

        # Directories for saving photos and videos
        self.photo_directory = "/home/paul/Desktop/final/photos"
        self.video_directory = "/home/paul/Desktop/final/videos"

        # Create the photos directory if it doesn't exist
        if not os.path.exists(self.photo_directory):
            os.makedirs(self.photo_directory)

        # Create the videos directory if it doesn't exist
        if not os.path.exists(self.video_directory):
            os.makedirs(self.video_directory)

        # Update video stream
        self.update_video()

    def update_video(self):
        try:
            # Stop the camera preview if it is running
            self.camera.stop_preview()
        except:
            pass

        self.camera.start_preview()
        sleep(2)  # Allow camera to warm up
        self.video_stream()

    def video_stream(self):
        try:
            stream = io.BytesIO()
            self.camera.capture(stream, format='jpeg', use_video_port=True)
            stream.seek(0)
            image = Image.open(stream)
            photo = ImageTk.PhotoImage(image)

            self.video_label.config(image=photo)
            self.video_label.image = photo

            self.master.after(10, self.video_stream)  # Update every 10 milliseconds
        except KeyboardInterrupt:
            self.camera.stop_preview()

    def capture_photo(self):
        photo_name = f"photo_{self.photo_counter}.jpg"
        photo_path = os.path.join(self.photo_directory, photo_name)
        self.camera.capture(photo_path)
        self.show_message(f"照片已拍攝並保存至 {photo_path}")

        # Increment photo counter
        self.photo_counter += 1

    def start_recording(self):
        video_name = f"video_{self.video_counter}.h264"
        video_path = os.path.join(self.video_directory, video_name)
        self.camera.start_recording(video_path)
        self.show_message(f"錄影已開始，檔案保存至 {video_path}")
        self.recording = True

        # Update button states
        self.record_button.config(state=tk.DISABLED)
        self.stop_record_button.config(state=tk.NORMAL)

    def stop_recording(self):
        self.camera.stop_recording()
        self.show_message("錄影已停止")
        self.recording = False

        # Update button states
        self.record_button.config(state=tk.NORMAL)
        self.stop_record_button.config(state=tk.DISABLED)

        # Increment video counter
        self.video_counter += 1

    def show_message(self, message):
        self.message_label.config(text=message)

if __name__ == "__main__":
    root = tk.Tk()
    app = CameraApp(root)
    root.mainloop()



```
請測試此程式是否能夠做出拍攝照片及錄影功能
### Step 6將picamera固定於pixycam上:
利用熱融槍或任何東西將picamera固定於pixycam之上並開啟pixymon和CameraApp對齊兩台相機之角度
### Step 7開始體驗自動追焦相機<img width="582" alt="截圖 2024-01-09 上午10 22 29" src="https://github.com/PaulSu0905/IoT-Final-Project-Documentation/assets/106649416/879c421d-13f1-4a86-b3b9-9e59963463da">

開始pixymon選定好要追蹤之物體<img width="582" alt="截圖 2024-01-09 上午10 23 30" src="https://github.com/PaulSu0905/IoT-Final-Project-Documentation/assets/106649416/76f2b9b4-57c8-4229-aeca-97cf58b41dc8">


開啟pan_tilt_demo開始追蹤物品
開啟CameraApp即可幫你想追焦的物品拍照或錄影



## demo Videos
https://youtube.com/shorts/sRNZN6Yw2D0?feature=shared




