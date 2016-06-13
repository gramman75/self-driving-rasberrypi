.. picamera_basic

================
Picamera 기본 기능
================


사진 촬영
=======


파일로 저장
--------

사진을 촬영하는 가장 기본적인 방법은 촬영한 이미지를 파일로 생성하는 방법입니다. 이때 사용하는 함수가 `capture() <https://picamera.readthedocs.io/en/release-1.10/api_camera.html#picamera.camera.PiCamera.capture>`_ 입니다.

.. code-block:: python

    import time
    import picamera

    with picamera.PiCamera() as camera:
        camera.resolution = (1024, 768)
        camera.start_preview()
        time.sleep(2)
        camera.capture('foo.jpg')

image의 format은 파일명의 확장자를 이용하여 판단합니다. 지원하지 않는 확장자일 경우 에러가 발생합니다.
또는 명시적으로 image의 format을 지정할 수 있습니다.::

    camera.capture('foo1.jpg',foramt='jpeg')

또 다른 parameter로 ``use_video_port`` 가 있습니다. 사진 촬영시 camera의 image port와 video port가 있는데
어느 것을 사용할지 결정하는 것 입니다. Default로 False값이며 image port를 사용합니다. 2 port의 차이점은 image port는 느리지만 품질이 더 좋고
video port는 빠른 촬영시 유용하지만 품질은 떨어 집니다.

Stream으로 저장
-------------

`Stream <https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%8A%B8%EB%A6%BC_(%EC%BB%B4%ED%93%A8%ED%8C%85)>`_ 은 파일을 읽거나 쓸 때, 네트워크 소켓을 거쳐 통신할 때 사용하는 추상 개념입니다.
python에서는 ``io`` 모듈을 이용합니다.

.. code-block:: python

    import io
    import time
    import picamera

    # Create an in-memory stream
    my_stream = io.BytesIO()
    with picamera.PiCamera() as camera:
        camera.start_preview()
        # Camera warm-up time
        time.sleep(2)
        camera.capture(my_stream, 'jpeg')

위의 예에서 my_stream을 이용해서 file에 저장할 수도 있고, socket을 통해 네트워크 전송에 사용할 수도 있습니다.

다음은 file stream을 통해서 저장하는 예제입니다. 일반적으로 file을 open한 후에 close를 하지 않으면 그 파일을 사용할 수가 없습니다.
하지만 capture를 이용하면, 그 순간 flush 함수가 수행이 되어 close하기 전에도 다른 프로세스에서 접근이 가능합니다.

.. code-block:: python

    import time
    import picamera

    my_file = open('image.jpg', 'wb')
    with picamera.PiCamera() as camera:
        camera.start_preview()
        time.sleep(2)
        camera.capture(my_file)
    # capture하는 순간 flush 함수가 수행이 되어 file을 close하기 전에도 사용할 수 있음
    my_file.close()

PIL로 저장
--------

PIL(Python Image Library)은 Python에서 Image를 편집하는 Libraray입니다. 하지만 2009년 11월 이후 Release가 되고 있지 않습니다.
가장 큰 문제는 Python 3.x를 지원하지 않는 다는 것입니다. 현재는 Pillow 라는 라이브러리가 PIL을 fork하여 프로젝트가 진행되고 있습니다. API가 거의 같기 때문에 pillow를 사용해도 괜찮습니다.

설치는 terminal에서 아래와 같이 입력하면 됩니다.::

    sudo pip install pillow

Compile이 진행이 완료가 되면 아래와 같이 picamera를 통해서 읽어 들입니다.

저장 방법은 Stream형태로 읽어 드린 후에 image의 시작점으로 이동하여 ``Image`` Module을
이용하여 생성하면 됩니다.

.. code-block:: python

    #-*- coding:utf-8-*-
    import io
    import time
    import picamera
    from PIL import Image

    my_stream = io.BytesIO()
    with picamera.PiCamera() as camera:
        camera.start_preview()
        time.sleep(2)
        camera.capture(my_stream, 'jpeg')

    my_stream.seek(0) # stream의 맨 앞으로 이동.
    image = Image.open(my_stream)
    image.save('bar.jpg') # image저장. pillow API를 적용할 수 있음.

OpenCV Object로 저장
------------------

다음은 stream으로 읽은 후에 openCV Object로 저장하는 방법입니다. rasberry pi에 OpenCV 3.1버전을 설치해야 합니다.
설치방법은 `이곳 <http://www.pyimagesearch.com/2015/02/23/install-opencv-and-python-on-your-raspberry-pi-2-and-b/>`_을
참고하시기 바랍니다. 소스를 직접 compile을 해야 하기 때문에 하루정도 시간이 소요가 됩니다.(전체하는데 3일정도 소요되는 것 같습니다.) 특히 openCV 소스를 Compile할 때는
background로 compile을 하는 것이 좋습니다.

설치가 완료가 되면 아래와 같이 stream으로 읽어 들인 후  numpy array로 변환하여 읽습니다.

.. code-block:: python

    #-*- coding:utf-8 -*-
    import io
    import time
    import picamera
    import cv2
    import numpy as np

    stream = io.BytesIO()
    with picamera.PiCamera() as camera:
        camera.start_preview()
        time.sleep(2)
        camera.capture(stream, format='jpeg')

    # stream을 numpy array로 변환.
    data = np.fromstring(stream.getvalue(), dtype=np.uint8)

    # numpy data를 image로 read.
    image = cv2.imdecode(data, 1)

    # OpenCV는 BGR로 읽어 들이기 때문에 ,RGB로 변환하기 위해서 3차원 값을 순서를 변경함.
    image = image[:, :, ::-1]

    ret, dst = cv2.threshold(image, 125, 255,cv2.THRESH_BINARY) # openCV의 threshold 적용

    cv2.imshow('img',dst)
    cv2.waitKey(0)
    cv2.destroyAllWindows()

위의 예에서 JPEG로 변환하여 다시 array로 변화하는 것을 피하고, 처리 속도를 높이기 위해서 ``picamera.array`` 모듈을 이용하는 것이 좋습니다.
그리고 openCV에서 사용하기 위해서 BGR로 format을 지정하면 됩니다.

.. code-block:: python

    # -*-codgin:utf-8 -*-
    import time
    import picamera
    import picamera.array
    import cv2

    with picamera.PiCamera() as camera:
        camera.start_preview()
        time.sleep(2)
        with picamera.array.PiRGBArray(camera) as stream:
            camera.capture(stream, format='bgr') # openCV에서 사용하기 위해 bgr로 읽음.

            image = stream.array

    cv2.imshow('image', image)
    cv2.waitKey(0)
    cv2.destroyAllWindows()

image resize하여 저장하기
----------------------

이미지를 분석하고 처리할 때, 초기 camera 해상도보다 작은 이미지로 변환이 필요할 때가 있습니다.
openCV나 pillow 라이브러리의 함수를 이용하는 방법보다 Raberry Pi의 GPU를 이용하여 resize하는 것이
효과적입니다. 아래 처럼 capture할 때 option으로 해상도를 지정하면 됩니다.

.. code-block:: python

    # -*-codgin:utf-8 -*-
    import time
    import picamera

    with picamera.PiCamera() as camera:
        camera.resolution = (1024, 768) # camera 초기 해상도.
        camera.start_preview()
        # Camera warm-up time
        time.sleep(2)
        camera.capture('foo.jpg', resize=(320, 240)) # 저장시 해상도 지정.

동일한 속성 이미지 저장하기
--------------------

연속된 사진을 찍을 때 동일한 밝기(brightness), 색깔(color), 색대비(contrast)를 유지해야할 때가 있습니다.
이를 위해서 연속사진을 찍을 때 사용할 수 있는 속성이 있습니다.

    * 노출시간을 고정시키기 위해서 ``shutter_speed``
    * 감도(감광 속도.ISO)를 고정하기 위해서 ``analog_gain`` , ``digital_gain`` . 이때 ``exposuer_mode`` 은 ``off`` 임.
    * White balance를 고정하기 위해서 ``awb_mode`` 를 ``off`` . ``awb_gains`` 에 (red, blue) tuple을 setting.
    * ISO를 고정하기 위해서 ``iso``

하지만, 일 속성에 적합한 값을 찾는 것은 어렵습니다. 예를 들면 ``iso`` 값은 낮에는 100 ~ 200, 빛이 적으면 400 ~ 500 이 적당합니다.
``shutter_speed`` 값은 ``exposure_speed`` 값을 구해서 지정하면 됩니다. 값을 고정한 다음에는 ``exposuer_mode`` 를 ``off`` 로 지정해야 합니다.

``awb_gains`` 값을 찾은 다음, ``awb_mode`` 를 ``off`` 로 설정하고, 구한 값은 다시 지정합니다.

아래는 이미지 속성 값을 고정한 예제입니다.

.. code-block:: python

    # -*-codgin:utf-8 -*-
    import time
    import picamera

    with picamera.PiCamera() as camera:
        camera.resolution = (1280, 720)
        camera.framerate = 30

        time.sleep(5)

        # shutter speed fix.
        camera.shutter_speed = camera.exposure_speed
        camera.exposure_mode = 'off'

        # white balance fix.
        g = camera.awb_gains
        camera.awb_mode = 'off'
        camera.awb_gains = g

        camera.capture_sequence(['image%02d.jpg' % i for i in range(10)])

결과는 image00.jpg ~ image09.jpg의 10장의 사진이 소스에서 고정한 동일한 값으로 저장이 됩니다.


timelapse