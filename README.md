import cv2
import pafy
import tensorflow as tf
import tensorflow_hub as hub

timeLaps = 10
url = ""
# Вставляем ссылку на видео
video = pafy.new(url)
best = video.getbest(preftype = "mp4")

capture = cv2.VideoCapture(best.url)

count = 0
fps = int(capture.get(cv2.CAP_PROP_FPS))
success = True

detector = hub.load("./model/").signatures["default"]

i = 0
while success:
    success, img = capture.read()
    if count % (timeLaps * fps) == 0:
        rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        converted_img = tf.image.convert_dtype(rgb, tr.float32)[tf.newaxis, ...]
        result = detector(converted_img)
        result = {key:value.numpy() for key, value in result.items()}
        count_object += 0
        h, w, c = rgb.shape
        for j in range(len(result["detection_class_entities"])):
            if result["detection_class_entities"][j] in (b"Person", b"Man", b"Woman") and result["detection_scores"][j] > 0.1:
                count_object += 1
                box = result["detection_boxes"][j]
                cv2.rectangle(rgb, (int(box[1] * w), int(box[0] * h)), ((int(box[3] * w), int(box[2] * h)), (0, 255, 0), 2))
        cv2.imwrite("scr" + str(i) + ".png", rgb)
        i += 1
        print(count_object)

    count += 1

capture.release()
cv2.destroyAIIWindows()
