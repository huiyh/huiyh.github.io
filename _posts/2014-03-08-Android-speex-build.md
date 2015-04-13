---
layout: post
category: android-development
title: Android中Speex音频编解码库的编译
description: ""
modified: 2014-02-24
tags: [android]
comments: true
share: true
image:
  feature: abstract-2.jpg
  credit: 
  creditlink: 
comments: true
---
[Speex](http://www.speex.org/)是一款开源的、高性能的音频编解码库，而且比较适合网络传输，需要语音网络传输的应用肯定要使用音频编解码来提高网络传输的效率，我们要把它应用在Android中的项目，首先要将speex编译成.so的动态链接库，再通过jni调用。这次笔者将介绍如何在Android中编译speex库。

* 先看一下基本的文件结构

![](https://github.com/baoyongzhang/test_pages/blob/gh-pages/image-9.png?raw=true)

* 新建一个jni目录，用于存放编译的文件以及*.mk文件，libs目录里面存放编译之后的*.so文件，不同目录对应不同的CPU架构，我们可以通过Application.mk文件配置需要编译的架构类型，添加如下内容。

{% highlight java %}

APP_ABI := armeabi armeabi-v7a x86 mips

{% endhighlight %}

* jni目录下的 include、libspeex是Speex库的文件，speex-jni是我们jni所需的“中介”文件，ndk方面的知识这里不再多说。进入正题，我们需要编写一个java类声明native方法用来调用so中的函数，直接上代码。

{% highlight java %}

package cc.itbox.testspeex.media;

/**
 * Speex音频编解码
 * @author Baoyz
 *
 */
public class Speex {

	/*
	 * quality 1 : 4kbps (very noticeable artifacts, usually intelligible) 
	 * 2 : 6kbps (very noticeable artifacts, good intelligibility)
	 * 4 : 8kbps (noticeable artifacts sometimes)
	 * 6 : 11kpbs (artifacts usually only noticeable with headphones) 
	 * 8 : 15kbps (artifacts not usually noticeable)
	 */
	private static final int DEFAULT_COMPRESSION = 4;

	public Speex() {
	}

	public void init() {
		load();
		open(DEFAULT_COMPRESSION);
	}

	private void load() {
		try {
			System.loadLibrary("speex_jni");
		} catch (Throwable e) {
			e.printStackTrace();
		}

	}

	public native int open(int compression);

	public native int getFrameSize();

	public native int decode(byte encoded[], short lin[], int size);

	public native int encode(short lin[], int offset, byte encoded[], int size);

	public native void close();

}

{% endhighlight %}

* 然后是speex_jni.cpp文件

{% highlight java %}

#include <jni.h>

#include <string.h>
#include <unistd.h>

#include <speex/speex.h>

static int codec_open = 0;

static int dec_frame_size;
static int enc_frame_size;

static SpeexBits ebits, dbits;
void *enc_state;
void *dec_state;

static JavaVM *gJavaVM;

extern "C"
JNIEXPORT jint JNICALL Java_cc_itbox_testspeex_media_Speex_open
  (JNIEnv *env, jobject obj, jint compression) {
    int tmp;

    if (codec_open++ != 0)
        return (jint)0;

    speex_bits_init(&ebits);
    speex_bits_init(&dbits);

    enc_state = speex_encoder_init(&speex_nb_mode);
    dec_state = speex_decoder_init(&speex_nb_mode);
    tmp = compression;
    speex_encoder_ctl(enc_state, SPEEX_SET_QUALITY, &tmp);
    speex_encoder_ctl(enc_state, SPEEX_GET_FRAME_SIZE, &enc_frame_size);
    speex_decoder_ctl(dec_state, SPEEX_GET_FRAME_SIZE, &dec_frame_size);

    return (jint)0;
}

extern "C"
JNIEXPORT jint Java_cc_itbox_testspeex_media_Speex_encode
    (JNIEnv *env, jobject obj, jshortArray lin, jint offset, jbyteArray encoded, jint size) {

        jshort buffer[enc_frame_size];
        jbyte output_buffer[enc_frame_size];
    int nsamples = (size-1)/enc_frame_size + 1;
    int i, tot_bytes = 0;

    if (!codec_open)
        return 0;

    speex_bits_reset(&ebits);

    for (i = 0; i < nsamples; i++) {
        env->GetShortArrayRegion(lin, offset + i*enc_frame_size, enc_frame_size, buffer);
        speex_encode_int(enc_state, buffer, &ebits);
    }
    //env->GetShortArrayRegion(lin, offset, enc_frame_size, buffer);
    //speex_encode_int(enc_state, buffer, &ebits);

    tot_bytes = speex_bits_write(&ebits, (char *)output_buffer,
                     enc_frame_size);
    env->SetByteArrayRegion(encoded, 0, tot_bytes,
                output_buffer);

        return (jint)tot_bytes;
}

extern "C"
JNIEXPORT jint JNICALL Java_cc_itbox_testspeex_media_Speex_decode
    (JNIEnv *env, jobject obj, jbyteArray encoded, jshortArray lin, jint size) {

        jbyte buffer[dec_frame_size];
        jshort output_buffer[dec_frame_size];
        jsize encoded_length = size;

    if (!codec_open)
        return 0;

    env->GetByteArrayRegion(encoded, 0, encoded_length, buffer);
    speex_bits_read_from(&dbits, (char *)buffer, encoded_length);
    speex_decode_int(dec_state, &dbits, output_buffer);
    env->SetShortArrayRegion(lin, 0, dec_frame_size,
                 output_buffer);

    return (jint)dec_frame_size;
}

extern "C"
JNIEXPORT jint JNICALL Java_cc_itbox_testspeex_media_Speex_getFrameSize
    (JNIEnv *env, jobject obj) {

    if (!codec_open)
        return 0;
    return (jint)enc_frame_size;

}

extern "C"
JNIEXPORT void JNICALL Java_cc_itbox_testspeex_media_Speex_close
    (JNIEnv *env, jobject obj) {

    if (--codec_open != 0)
        return;

    speex_bits_destroy(&ebits);
    speex_bits_destroy(&dbits);
    speex_decoder_destroy(dec_state);
    speex_encoder_destroy(enc_state);
}

{% endhighlight %}

* 准备好后，使用ndk-build进行编译。

* [Speex编译Demo代码下载](https://github.com/ITBox/TestSpeexBuild)
