#!/usr/bin/env python
# coding=utf-8


#import sys
import rospy
import cv2
#import cv
from baxter_interface import (
     CameraController


    )

import argparse
import numpy as np
#from std_msgs.msg import String
from sensor_msgs.msg import Image
from cv_bridge import CvBridge, CvBridgeError
import std_srvs.srv


class ColorTest ():

    def __init__(self):
        self.cam ="/cameras/left_hand_camera/image"

        self.image_pub=rospy.Publisher("image_topic_2",Image)

        self.disp_pub=rospy.Publisher('/robot/xdisplay', Image, latch=True)#Pub to xdisplay, but have to us cv_brige to convert befor pub

        self.image_sub=rospy.Subscriber(self.cam,Image,self.image_callback)
        
        self.bridge=CvBridge()

        self.left_camera=CameraController("left_hand_camera")
        self.left_camera.resolution=(640,400)

        self.reset_cameras()
        self.left_camera.close()
        self.left_camera.open()

        

########################################################################################
#    def color_mask (self,ball_color):
#        lower = (86,31,4)
#        upper = (220,88,50)
 #       if ball_color == "blue":
 #           lower = (86,31,4)
 #           upper = (220,88,50)
#
 #       if ball_color == "green":
#            lower = (29,86,6)
 #           upper = (64,255,255)
#
 #       if ball_color == "red":
 #           lower = (17,15,100)
#            upper = (50,56,200)

  #      image_hsv = cv2.cvtColor(image_src,cv2.COLOR_BGR2HSV)##################
  #      image_mask = cv2.inRange(image_hsv,lower,upper)

########################################################################################
    def reset_cameras (self):
        reset_srv = rospy.ServiceProxy('cameras/reset', std_srvs.srv.Empty)
        rospy.wait_for_service('cameras/reset', timeout=10)
        reset_srv()


    def disp_img(self,image):
        msg=CvBridge().cv2_to_imgmsg(image, encoding="mono16")
        self.disp_pub.publish(msg)
##################################################################################

    def image_callback (self,data):
        print "image_callbacking........./n"

        #converting the image message to opencv message
        try:
            #image_cv = self.bridge.imgmsg_to_cv2 (data,"bgr8")
            image_cv=np.squeeze(np.array(self.bridge.imgmsg_to_cv2(data, "passthrough")))
        except CvBridgeError,e:
            print e

        
        if image_cv is not None :

            print "get the image_cv...../n "
            height, width = image_cv.shape[:2]
            #print ("/n{0}",format(height))
            #print "----------------------"
            #print ("/n{0}",format(width))
            #print "----------------------"













        #converting the opencv msg to image msg then publish
        try:
            self.image_pub.publish(self.bridge.cv2_to_imgmsg(image_cv, "bgr8"))
        except CvBridgeError as e:
            print(e)

        print "finished image_callback........"

##################################################################################

def main():
    global ball_color
    
    
    print ("Initializing...........................................")
    rospy.init_node("ylj_camTest",anonymous=True)

    arg_fmt = argparse.RawDescriptionHelpFormatter
    parser = argparse.ArgumentParser(formatter_class=arg_fmt, description=main.__doc__)

    parser.add_argument(
        '-c','---color', dest='color',choices=['blue','green','red'], required=True,
        help= "the ball color to pick up", 
         )

    args=parser.parse_args()
    ball_color = args.color
######################################################################################
    ct = ColorTest()
    print "system is start"
    try:
        rospy.spin()
    except KeyboardInterrupt:
        print "Shutting down"


    cv2.destroyAllWindows()

if __name__=='__main__':

    main()
