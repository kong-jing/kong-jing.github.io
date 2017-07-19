---
layout: post
title: MediaCodec视频文件硬件解码为YUV格式帧
date: 2017-07-19 16:02:41
tags: Android
categories: post
---



最近在Android设备上调用IPC网络视频资源的时候，由于摄像头是传递的硬解码的H264格式的文件，所以在Android设备上使用的时候颜色不合适。需要将视频帧转换为Android需要的YUV格式。

![mediacodec](/img/mediacodec.jpg)



**正常初始化MediaCode并启动解码器是下面这段代码**

```java
mMediaCodec.configure(mediaFormat, null, null, 0);

mMediaCodec.start();
```



**初始化MediaFormat之前得到的视频编码信息,指定视频帧格式要添加下面代码**

```java
try {
mMediaCodec = MediaCodec.createDecoderByType(MIME_TYPE);
	} catch (IOException e) {
	e.printStackTrace();
}
mediaFormat = MediaFormat.createVideoFormat(MIME_TYPE, info.width,
					info.height);
/**指定视频帧格式*/
mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT,MediaCodecInfo.CodecCapabilities.COLOR_FormatYUV420Flexible);//指定帧格式
/***/
mMediaCodec.configure(mediaFormat, null, null, 0);
```

对于支持`video/avc`的解码器，支持的帧格式是*COLOR_FormatYUV420Flexible*



**获取视频流**

```java
byte arr[] = GetCameraStreamJNI.get().GetImage(timeout_ms);//获取视频流
if (arr[4] != 103 && arr[4] != 101) {//获取视频的 关键i帧
					  continue;
					}
```

**指定Image来转换帧格式**

```java
/**指定转换帧格式*/
			Image image = mMediaCodec.getOutputImage(outputBufferIndex);
			outData = getDataFromImage(image, 2);//指定解码帧为 NV21
			image.close();
/***/
```

**获取YUV420_888编码后，将编码Image转换为I420（标准YUV）和NV21格式byte数组**

```java
private static boolean isImageFormatSupported(Image image) {
		int format = image.getFormat();
		switch (format) {
			case ImageFormat.YUV_420_888:
			case ImageFormat.NV21:
			case ImageFormat.YV12:
				return true;
		}
		return false;
	}

private static byte[] getDataFromImage(Image image,int colorFormat) {

		if (!isImageFormatSupported(image)) {
			throw new RuntimeException("can't convert Image to byte array, format " + image.getFormat());
		}
		Rect crop = image.getCropRect();
		int format = image.getFormat();
		int width = crop.width();
		int height = crop.height();
		Image.Plane[] planes = image.getPlanes();
		byte[] data = new byte[width * height * ImageFormat.getBitsPerPixel(format) / 8];
		byte[] rowData = new byte[planes[0].getRowStride()];
		int channelOffset = 0;
		int outputStride = 1;
		for (int i = 0; i < planes.length; i++) {
			switch (i) {
				case 0:
					channelOffset = 0;
					outputStride = 1;
					break;
				case 1:
					if (colorFormat == 1) {//YUV_420_888
						channelOffset = width * height;
						outputStride = 1;
					} else if (colorFormat == 2) {//NV21
						channelOffset = width * height + 1;
						outputStride = 2;
					}
					break;
				case 2:
					if (colorFormat == 1) {//YUV_420_888
						channelOffset = (int) (width * height * 1.25);
						outputStride = 1;
					} else if (colorFormat == 2) {//NV21
						channelOffset = width * height;
						outputStride = 2;
					}
					break;
			}
			ByteBuffer buffer = planes[i].getBuffer();
			int rowStride = planes[i].getRowStride();
			int pixelStride = planes[i].getPixelStride();

			int shift = (i == 0) ? 0 : 1;
			int w = width >> shift;
			int h = height >> shift;
			buffer.position(rowStride * (crop.top >> shift) + pixelStride * (crop.left >> shift));
			for (int row = 0; row < h; row++) {
				int length;
				if (pixelStride == 1 && outputStride == 1) {
					length = w;
					buffer.get(data, channelOffset, length);
					channelOffset += length;
				} else {
					length = (w - 1) * pixelStride + 1;
					buffer.get(rowData, 0, length);
					for (int col = 0; col < w; col++) {
						data[channelOffset] = rowData[col * pixelStride];
						channelOffset += outputStride;
					}
				}
				if (row < h - 1) {
					buffer.position(buffer.position() + rowStride - length);
				}
			}
			 Log.v(TAG, "Finished reading data from plane " + i);
		}
		return data;
	}
```

**之后就能够使用NV21来转换为bitmap**

这里有一个NV21转化为bitmap的方法。

```java
private Bitmap yuv2Img(byte[] frameData, int yuvFormat, int prevWidth, int prevHeight, int quality) {
    Long start = System.currentTimeMillis();
    Bitmap img = null;
    try {
      YuvImage image = new YuvImage(frameData, yuvFormat, prevWidth, prevHeight, null);
      if (image != null) {
        ByteArrayOutputStream stream = new ByteArrayOutputStream();
        image.compressToJpeg(new Rect(0, 0, prevWidth, prevHeight), quality, stream);
        img = BitmapFactory.decodeByteArray(stream.toByteArray(), 0, stream.size());

        stream.close();
      }
    } catch (Exception ex) {
      LogUtil.LOGE(TAG, "yuv2Img异常:" + ex.getMessage());
    }
    LogUtil.LOGE(TAG, "yuv2Img" + (System.currentTimeMillis() - start));
    return img;
  }
```



