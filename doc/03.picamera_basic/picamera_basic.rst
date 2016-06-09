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

PIL