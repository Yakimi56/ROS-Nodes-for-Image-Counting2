# ROS-Nodes-for-Image-Counting
//Image Publisher Node (image_publisher_node.py):
#!/usr/bin/env python3 //

import rospy
from sensor_msgs.msg import Image
import cv2
from cv_bridge import CvBridge

class ImagePublisherNode:
    def __init__(self):
        rospy.init_node('image_publisher_node', anonymous=True)
        self.image_topic = "/cam_img"
        self.image_publisher = rospy.Publisher(self.image_topic, Image, queue_size=10)
        self.cv_bridge = CvBridge()
        self.webcam = cv2.VideoCapture(0)

    def publish_images(self):
        rate = rospy.Rate(5)  # 5 Hz publishing rate
        while not rospy.is_shutdown():
            ret, frame = self.webcam.read()
            if ret:
                image_msg = self.cv_bridge.cv2_to_imgmsg(frame, "bgr8")
                self.image_publisher.publish(image_msg)
            rate.sleep()

if __name__ == '__main__':
    try:
        publisher_node = ImagePublisherNode()
        publisher_node.publish_images()
    except rospy.ROSInterruptException:
        pass

        
//Image Counter Node (image_counter_node.py):
#!/usr/bin/env python3//

import rospy
from std_msgs.msg import Empty, String
from sensor_msgs.msg import Image
import cv2
import os
import math
from cv_bridge import CvBridge

class ImageCounterNode:
    def __init__(self):
        rospy.init_node('image_counter_node', anonymous=True)
        self.image_topic = "/cam_img"
        self.image_subscriber = rospy.Subscriber(self.image_topic, Image, self.image_callback, queue_size=10)
        self.reset_subscriber = rospy.Subscriber("/reset", Empty, self.reset_callback)
        self.counter = 0
        self.cv_bridge = CvBridge()

    def image_callback(self, image_msg):
        self.counter += 1
        if self.is_prime(self.counter):
            image = self.cv_bridge.imgmsg_to_cv2(image_msg, "bgr8")
            image_path = self.save_image(image)
            self.publish_image_path(image_path)

    def is_prime(self, number):
        if number < 2:
            return False
        for i in range(2, int(math.sqrt(number)) + 1):
            if number % i == 0:
                return False
        return True

    def save_image(self, image):
        file_name = f"image_{self.counter}.jpg"
        file_path = os.path.join(os.path.expanduser("~"), "saved_images", file_name)
        os.makedirs(os.path.dirname(file_path), exist_ok=True)
        cv2.imwrite(file_path, image)
        return file_path

    def publish_image_path(self, image_path):
        path_publisher = rospy.Publisher("/img_path", String, queue_size=10)
        path_publisher.publish(image_path)

    def reset_callback(self, _):
        self.counter = 0

if __name__ == '__main__':
    try:
        counter_node = ImageCounterNode()
        rospy.spin()
    except rospy.ROSInterruptException:
        pass
//Image Path Saver Node (image_path_saver_node.py):
#!/usr/bin/env python3//

import rospy
from std_msgs.msg import String
import os

class ImagePathSaverNode:
    def __init__(self):
        rospy.init_node('image_path_saver_node', anonymous=True)
        self.image_path_topic = "/img_path"
        self.image_path_subscriber = rospy.Subscriber(self.image_path_topic, String, self.image_path_callback, queue_size=10)
        self.reset_subscriber = rospy.Subscriber("/reset", Empty, self.reset_callback)
        self.output_file_path = os.path.join(os.path.expanduser("~"), "image_paths.txt")

    def image_path_callback(self, image_path_msg):
        with open(self.output_file_path, 'a') as f:
            f.write(image_path_msg.data + '\n')

    def reset_callback(self, _):
        with open(self.output_file_path, 'w') as f:
            f.write("")

if __name__ == '__main__':
    try:
        path_saver_node = ImagePathSaverNode()
        rospy.spin()
    except rospy.ROSInterruptException:
        pass
//Run the Image Publisher Node://
$ chmod +x image_publisher_node.py
$ rosrun your_package_name image_publisher_node.py
//Run the Image Counter Node://
$ chmod +x image_counter_node.py
$ rosrun your_package_name image_counter_node.py
//Run the Image Path Saver Node://
$ chmod +x image_path_saver_node.py
$ rosrun your_package_name image_path_saver_node.py
//To reset the counter and clear the image path file, open a new terminal and publish an empty message to the "/reset" topic://
$ rostopic pub /reset std_msgs/Empty "{}"
