---
title: 音视频解码-MediaCodec
date: 2019.12.04 17:01:00
tags: Android
categories: Android
---


MediaExtractor和MediaCodec:
MediaExtractor：读取解析音视频文件的轨道数据和基本信息
MediaCodec:将音视频文件解码成pcm原生数据

**解析音频实例**
Mp3Converter.java
```
private void prepareDecode(String path) {
    try {
        mediaExtractor = new MediaExtractor();
        mediaExtractor.setDataSource(path);

        File file = new File(path);
        if(file.exists()){
            fileSize = file.length();
        }

        for (int i = 0; i < mediaExtractor.getTrackCount(); i++) {       //获取轨道数量
            MediaFormat mMediaFormat = mediaExtractor.getTrackFormat(i); //获取编号轨道格式
            Log.i(TAG, "prepareDecode get mMediaFormat=" + mMediaFormat);

            String mime = mMediaFormat.getString(MediaFormat.KEY_MIME);  //获取格式
            if (mime.startsWith("audio")) {
                mMediaCodec = MediaCodec.createDecoderByType(mime);
                mMediaCodec.configure(mMediaFormat, null, null, 0);      //将mediaExtractor配置注入mMediaCodec（第一个参数是待解码的数据格式(也可用于编码操作);第二个参数是设置surface，用来在其上绘制解码器解码出的数据；第三个参数于数据加密有关；第四个参数上1表示编码器）
                mediaExtractor.selectTrack(i); //切换到该轨道(这样在之后调用readSampleData()/getSampleTrackIndex()方法时候，返回的就只是该轨道的数据了，其他轨的数据不会被返回)
                inSampleRate = mMediaFormat.getInteger(MediaFormat.KEY_SAMPLE_RATE); //音频格式的采样率
                channels = mMediaFormat.getInteger(MediaFormat.KEY_CHANNEL_COUNT);   //音频格式的频道数量
                break;
            }
        }
        mMediaCodec.start(); //启动MediaCodec

        bufferInfo = new MediaCodec.BufferInfo();
        if (android.os.Build.VERSION.SDK_INT < android.os.Build.VERSION_CODES.LOLLIPOP) {
            rawInputBuffers = mMediaCodec.getInputBuffers();
            encodedOutputBuffers = mMediaCodec.getOutputBuffers();
        }
        Log.i(TAG,  "--channel=" + channels + "--sampleRate=" + inSampleRate);

        mp3Lame = new Mp3Lame();
        mp3Lame.init(inSampleRate, channels, inSampleRate, 128, 9);
    } catch (IOException e) {
        e.printStackTrace();
    }
}

private void readSampleData() {
    boolean rawInputEOS = false;
    if(android.os.Build.VERSION.SDK_INT < android.os.Build.VERSION_CODES.LOLLIPOP){
        while (!rawInputEOS) {
            for (int i = 0; i < rawInputBuffers.length; i++) {
                int inputBufIndex = mMediaCodec.dequeueInputBuffer(-1); // 得到可写入的buffer的索引
                if (inputBufIndex >= 0) {
                    ByteBuffer buffer = rawInputBuffers[inputBufIndex];
                    int sampleSize = mediaExtractor.readSampleData(buffer, 0); // 把MediaExtractor中的数据写入到这个可用的ByteBuffer对象中去，返回值为-1表示MediaExtractor中数据已全部读完
                    long presentationTimeUs = 0;
                    if (sampleSize < 0) {
                        rawInputEOS = true;
                        sampleSize = 0;
                    } else {
                        readSize += sampleSize;
                        presentationTimeUs = mediaExtractor.getSampleTime();
                    }
                    // 把buffer传递给解码器, 最初传递给解码器的buffer必须是带有BUFFER_FLAG_CODEC_CONFIG下标的编码特定数据
                    mMediaCodec.queueInputBuffer(inputBufIndex, 0,
                            sampleSize, presentationTimeUs,
                            rawInputEOS ? MediaCodec.BUFFER_FLAG_END_OF_STREAM : 0);
                    if (!rawInputEOS) {
                        mediaExtractor.advance(); // 在MediaExtractor执行完一次readSampleData方法后，需要调用advance()去跳到下一个sample，然后再次读取数据
                    } else {
                        break;
                    }
                } else {
                    Log.e(TAG, "wrong inputBufIndex=" + inputBufIndex);
                }
            }
        }
    }
    else{
        while (!rawInputEOS) {
            int inputBufIndex = mMediaCodec.dequeueInputBuffer(-1); // 得到可写入的buffer的索引
            if (inputBufIndex >= 0) {
                ByteBuffer buffer = null;
                buffer = mMediaCodec.getInputBuffer(inputBufIndex); // 根据索引获取buffer容器
                int sampleSize = mediaExtractor.readSampleData(buffer, 0); // 把MediaExtractor中的数据写入到这个可用的ByteBuffer容器中去，返回值为-1表示MediaExtractor中数据已全部读完
                long presentationTimeUs = 0;
                if (sampleSize < 0) {
                    rawInputEOS = true;
                    sampleSize = 0;
                } else {
                    readSize += sampleSize;
                    presentationTimeUs = mediaExtractor.getSampleTime();
                }
                // 把buffer容器传递回解码器, 最初传递给解码器的buffer必须是带有BUFFER_FLAG_CODEC_CONFIG下标的编码特定数据
                mMediaCodec.queueInputBuffer(inputBufIndex, 0,
                        sampleSize, presentationTimeUs,
                        rawInputEOS ? MediaCodec.BUFFER_FLAG_END_OF_STREAM : 0);
                if (!rawInputEOS) {
                    mediaExtractor.advance(); // 在MediaExtractor执行完一次readSampleData方法后，需要调用advance()去跳到下一个sample，然后再次读取数据
                } else {
                    break;
                }
            } else {
                Log.e(TAG, "wrong inputBufIndex=" + inputBufIndex);
            }
        }
    }
}

private class DecodeThread extends Thread {
    @Override
    public void run() {
        long startTime = System.currentTimeMillis();

        if(android.os.Build.VERSION.SDK_INT < android.os.Build.VERSION_CODES.LOLLIPOP){
            try {
                Log.i(TAG, "DecodeThread start");
                int num = 0;
                while (true) {
                    int outputBufIndex = mMediaCodec.dequeueOutputBuffer(bufferInfo, -1); // 获取解码后的数据的索引
                    if (outputBufIndex >= 0) {
                        ByteBuffer buffer = encodedOutputBuffers[outputBufIndex];
                        decodeSize += bufferInfo.size;
                        // 创建此字节缓冲区的视图，作为新的 short 缓冲区，新缓冲区的位置将为零，其容量和界限将为此缓冲区中所剩余的字节数的二分之一
                        ShortBuffer shortBuffer = buffer.order(ByteOrder.nativeOrder()).asShortBuffer();// nativeOrder()返回当前硬件平台的字节序

                        short[] leftBuffer = null;
                        short[] rightBuffer = null;
                        short[] pcm = null;

                        if (channels == 2) { //双轨道
                            pcm = new short[shortBuffer.remaining()]; //shortBuffer.remaining()返回剩下的buffer
                            shortBuffer.get(pcm); //读取给定索引处的 shortBuffer。
                        } else {             //单轨道
                            leftBuffer = new short[shortBuffer.remaining()];
                            rightBuffer = leftBuffer;
                            shortBuffer.get(leftBuffer); //读取给定索引处的 shortBuffer。
                            Log.e(TAG, "single channel leftBuffer.length = " + leftBuffer.length);
                        }

                        buffer.clear();

                        //用解码的数据构建BufferDecoded
                        BufferDecoded bufferDecoded = new BufferDecoded();
                        bufferDecoded.leftBuffer = leftBuffer;
                        bufferDecoded.rightBuffer = rightBuffer;
                        bufferDecoded.pcm = pcm;
                        bufferDecoded.channels = channels;
                        bufferDecoded.pts = bufferInfo.presentationTimeUs;
                        encodeQueue.put(bufferDecoded);

                        mMediaCodec.releaseOutputBuffer(outputBufIndex, false); //使用完成后，释放Buffer容器

                        if ((bufferInfo.flags & MediaCodec.BUFFER_FLAG_END_OF_STREAM) != 0) {
                            Log.e(TAG, "DecodeThread get BUFFER_FLAG_END_OF_STREAM");
                            decodeFinished = true;
                            break;
                        }
                    } else if (outputBufIndex == MediaCodec.INFO_OUTPUT_BUFFERS_CHANGED) {
                        encodedOutputBuffers = mMediaCodec.getOutputBuffers();
                    } else if (outputBufIndex == MediaCodec.INFO_OUTPUT_FORMAT_CHANGED) { //解码开始会执行一次, 数据流会遵循新的格式
                        final MediaFormat oformat = mMediaCodec.getOutputFormat();
                        Log.d(TAG, "MediaCodec.INFO_OUTPUT_FORMAT_CHANGED " + num);
                        Log.d(TAG, "Output format has changed to " + oformat);
                    }
                    num++;
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        else{
            try {
                Log.i(TAG, "DecodeThread start");
                int num = 0;
                while (true) {
                    int outputBufIndex = mMediaCodec.dequeueOutputBuffer(bufferInfo, -1); // 获取解码后的数据的索引
                    if (outputBufIndex >= 0) {
                        ByteBuffer buffer = mMediaCodec.getOutputBuffer(outputBufIndex);  //获取存有已解码数据的buffer容器
                        decodeSize += bufferInfo.size;
                        // 创建此字节缓冲区的视图，作为新的 short 缓冲区，新缓冲区的位置将为零，其容量和界限将为此缓冲区中所剩余的字节数的二分之一
                        ShortBuffer shortBuffer = buffer.order(ByteOrder.nativeOrder()).asShortBuffer();// nativeOrder()返回当前硬件平台的字节序

                        short[] leftBuffer = null;
                        short[] rightBuffer = null;
                        short[] pcm = null;

                        if (channels == 2) { //双轨道
                            pcm = new short[shortBuffer.remaining()]; //shortBuffer.remaining()返回剩下的buffer
                            shortBuffer.get(pcm); //读取给定索引处的 shortBuffer。
                        } else {             //单轨道
                            leftBuffer = new short[shortBuffer.remaining()];
                            rightBuffer = leftBuffer;
                            shortBuffer.get(leftBuffer); //读取给定索引处的 shortBuffer。
                            Log.e(TAG, "single channel leftBuffer.length = " + leftBuffer.length);
                        }

                        buffer.clear();

                        //用解码的数据构建BufferDecoded
                        BufferDecoded bufferDecoded = new BufferDecoded();
                        bufferDecoded.leftBuffer = leftBuffer;
                        bufferDecoded.rightBuffer = rightBuffer;
                        bufferDecoded.pcm = pcm;
                        bufferDecoded.channels = channels;
                        bufferDecoded.pts = bufferInfo.presentationTimeUs;
                        encodeQueue.put(bufferDecoded);

                        mMediaCodec.releaseOutputBuffer(outputBufIndex, false); //使用完成后，释放Buffer容器

                        if ((bufferInfo.flags & MediaCodec.BUFFER_FLAG_END_OF_STREAM) != 0) {
                            Log.e(TAG, "DecodeThread get BUFFER_FLAG_END_OF_STREAM");
                            decodeFinished = true;
                            break;
                        }
                    } else if (outputBufIndex == MediaCodec.INFO_OUTPUT_BUFFERS_CHANGED) {
                        encodedOutputBuffers = mMediaCodec.getOutputBuffers();
                    } else if (outputBufIndex == MediaCodec.INFO_OUTPUT_FORMAT_CHANGED) { //解码开始会执行一次, 数据流会遵循新的格式
                        final MediaFormat oformat = mMediaCodec.getOutputFormat();
                        Log.d(TAG, "MediaCodec.INFO_OUTPUT_FORMAT_CHANGED " + num);
                        Log.d(TAG, "Output format has changed to " + oformat);
                    }
                    num++;
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        encodeThread.interrupt();
        long endTime = System.currentTimeMillis();
        Log.i(TAG, "DecodeThread finished time=" + (endTime - startTime) / 1000);
    }
}
```

  MediaCodec还可以处理视频解码，由此也接触到十分强大的C类库FFmpeg。基于此库能实现音视频剪切，合成，转码，混音，水印，滤镜，转gif乃至直播处理等功能。有机会一定要研究下orz.

参考资料：
硬编码之MediaCodec:https://www.jianshu.com/p/f116b6f81ab3
MediaCodec文档翻译-https://www.cnblogs.com/Xiegg/p/3428529.html
