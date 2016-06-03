.. picamera_quick

===============
Picamera 사용법
===============

Python에서 picamera를 control하기 위해서는 picamera module이 설치가 되어 있어야 합니다. 기본적으로 rasbien을 사용하고 있으면 설치가 되어 있습니다.
다른 OS를 설치를 했다면 아래와 같이 입력하여 설치를 해야 합니다.::

    sudo pip install picamera

아래는 10초간 camera를 통한 영상을 보여주는 예제입니다.

.. code-block:: python

    import time
    import picamera

    camera = picamera.PiCamera()
    try:
        camera.start_preview()
        time.sleep(10)
        camera.stop_preview()
    finally:
        camera.close()

위 예제에서 중요한 부분은 camera를 사용하면 꼭 ``camera.close()`` 를 수행하여 resouce를 반납해야 합니다.
아래 예제는 python은 ``with/as`` 문을 이용하여 close를 암묵적으로 처리하는 예제입니다.

.. code-block:: python

    import time
    import picamera

    with picamera.PiCamera() as camera:
        camera.start_preview()
        time.sleep(10)
        camera.stop_preview()

``with/as`` 는 python에서 context를 관리하는 용도로 사용이 됩니다. 특정 class를 사용하고 종료가 되면 자동으로 지정된 함수가 수행이 됩니다.
예를 들면 file을 open한 후에 close를 해야 하는 경우, lock을 잡고 task가 끝나면 unlock을 해야하는 것을 예로 들 수 있습니다.

try/finally와 유사하지만 간단하게 표현되는 장점이 있습니다. 물론 with/as가 지원이 되도록 해당 class를 구현해야 합니다.



