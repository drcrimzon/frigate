# Frigate - Realtime Object Detection for RTSP Cameras
**Note:** This version requires the use of a [Google Coral USB Accelerator](https://coral.withgoogle.com/products/accelerator/)  
**Note2:** This version is ported from the Docker container version.  This setup is configured to run on Arch, using the latest libraries.  

Uses OpenCV and Tensorflow to perform realtime object detection locally for RTSP cameras. Designed for integration with HomeAssistant or others via MQTT.

- Leverages multiprocessing and threads heavily with an emphasis on realtime over processing every frame
- Allows you to define specific regions (squares) in the image to look for objects
- No motion detection (for now)
- Object detection with Tensorflow runs in a separate thread
- Object info is published over MQTT for integration into HomeAssistant as a binary sensor
- An endpoint is available to view an MJPEG stream for debugging

![Diagram](diagram.png)

## Example video (from older version)
You see multiple bounding boxes because it draws bounding boxes from all frames in the past 1 second where a person was detected. Not all of the bounding boxes were from the current frame.
[![](http://img.youtube.com/vi/nqHbCtyo4dY/0.jpg)](http://www.youtube.com/watch?v=nqHbCtyo4dY "Frigate")

## Getting Started  
yay -S python3 \  
pip \  
protobuf \  
opencv \  
edgetpu

pip install -U pip \  
numpy \  
pillow \  
matplotlib \  
notebook \  
Flask \  
imutils \  
paho-mqtt \  
PyYAML \  
ffmpeg

GIT_SSL_NO_VERIFY=true sudo git clone -q https://github.com/tensorflow/models /usr/local/lib/python3.5/dist-packages/tensorflow/models  
wget http://storage.googleapis.com/cloud-iot-edge-pretrained-models/edgetpu_api.tar.gz   
tar xvf edgetpu_api.tar.gz  
ln -s ./python-tflite-source/edgetpu/test_data/mobilenet_ssd_v2_coco_quant_postprocess_edgetpu.tflite /opt/frigate/config/forzen_inference_graph.pb  
ln -s ./python-tflite-source/edgetpu/test_data/coco_labels.txt /opt/frigate/config/label_map.pbtext  
  
export PYTHONPATH=/usr/local/lib/python3.5/dist-packages/tensorflow/models/research:/usr/local/lib/python3.5/dist-packages/tensorflow/models/research/slim  
cd /usr/local/lib/python3.5/dist-packages/tensorflow/models/research && sudo protoc object_detection/protos/*.proto --python_out=.  
  
A `config.yml` file must exist in the `config` directory. See example [here](config/config.yml).

Access the mjpeg stream at `http://localhost:5000/<camera_name>` and the best person snapshot at `http://localhost:5000/<camera_name>/best_person.jpg`

## Integration with HomeAssistant
```
camera:
  - name: Camera Last Person
    platform: generic
    still_image_url: http://<ip>:5000/<camera_name>/best_person.jpg

binary_sensor:
  - name: Camera Person
    platform: mqtt
    state_topic: "frigate/<camera_name>/objects"
    value_template: '{{ value_json.person }}'
    device_class: motion
    availability_topic: "frigate/available"
```

## Tips
- Lower the framerate of the RTSP feed on the camera to reduce the CPU usage for capturing the feed

## Future improvements
- [x] Remove motion detection for now
- [x] Try running object detection in a thread rather than a process
- [x] Implement min person size again
- [x] Switch to a config file
- [x] Handle multiple cameras in the same container
- [ ] Attempt to figure out coral symlinking
- [ ] Add object list to config with min scores for mqtt
- [ ] Move mjpeg encoding to a separate process
- [ ] Simplify motion detection (check entire image against mask, resize instead of gaussian blur)
- [ ] See if motion detection is even worth running
- [ ] Scan for people across entire image rather than specfic regions
- [ ] Dynamically resize detection area and follow people
- [ ] Add ability to turn detection on and off via MQTT
- [ ] Output movie clips of people for notifications, etc.
- [ ] Integrate with homeassistant push camera
- [ ] Merge bounding boxes that span multiple regions
- [ ] Implement mode to save labeled objects for training
- [ ] Try and reduce CPU usage by simplifying the tensorflow model to just include the objects we care about
- [ ] Look into GPU accelerated decoding of RTSP stream
- [ ] Send video over a socket and use JSMPEG
- [x] Look into neural compute stick
