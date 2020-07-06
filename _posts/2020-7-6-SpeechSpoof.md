---
layout: post
title: 声纹转换项目进度1
categories: Project
description: 声纹转换项目进度
keywords: Project,Spoof
---

## 项目介绍：

该项目研究目标，实现用户语音音色转换成他人音色，通俗理解例如：柯南的变声器。

## 当前研究内容：

当下对于该功能的实现方法较常见的为：将音频转化为mel-spectrogram，再将mel-spectrogram(以下简称mel)转化为linear-spectrogram(亦称为magnitude spectrogram，以下简称mag)，通过GAN迭代寻训练使mel及mag与目标发音相似，最终通过griffin_lim算法实现频谱特征转化成时域特征(即真实音频)

## 本次研究内容：

检验音频->mel->mag->音频过程是否能够真实还原原声。

### 研究内容如下：

本次研究主要采用[Spoof](https://arxiv.org/abs/1910.13054?context=eess)论文提供的源码进行测试

#### 1. librosa实现该过程

代码：

```python
import numpy,wave
import numpy as np
import torch
import librosa
import librosa.display
import matplotlib.pyplot as plt
import os
import pylab
from scipy import signal

'''音频转mel'''
#testpath="../speech/A2_2.wav"
testpath="test.wav"
speech, sr = librosa.core.load(path=testpath, sr=None, mono=True)
speech, _ = librosa.effects.trim(speech, 22)
speech = np.append(speech[0], speech[1:] - 0.97*speech[:-1])
print(torch.from_numpy(speech).shape)
lin_spec = np.abs(librosa.stft(y=speech, n_fft=1024, hop_length=256))
mel_filterbank = librosa.filters.mel(sr=sr, n_fft=1024, n_mels=80)
mel_spec = np.dot(mel_filterbank, lin_spec)

maxlin = np.max(lin_spec)
maxmel = np.max(mel_spec)

#normalization
lin_spec_norm = (lin_spec / maxlin)**0.6
mel_spec_norm = (mel_spec / maxmel)**0.6

reduced_total_time = np.shape(mel_spec)[1] // 4
#print(reduced_total_time)
reduced_time = [4*k for k in range(int(reduced_total_time))]
reduced_mel_spec = mel_spec_norm[:, reduced_time]
lin_spec_norm = lin_spec_norm[:, :4*reduced_total_time]


mel=torch.from_numpy(reduced_mel_spec)      #mel特征
lin=torch.from_numpy(lin_spec_norm)         #linear特征
# print(mel.shape)
# print(lin)

#linear转音频
spec = lin_spec_norm[:,:]**(1.3/0.6)
print(spec)
time_signal = librosa.core.griffinlim(S=spec, n_iter=64, hop_length=256, win_length=1024)
time_signal = signal.lfilter([1], [1, -0.97], time_signal)
#print(time_signal, np.max(time_signal))
librosa.output.write_wav('test1.wav', time_signal/np.max(time_signal)*0.75, 22050)

#绘制mel谱图与线性谱图
save_path = 'test_mel.png'
pylab.axis('off') # no axis
pylab.axes([0., 0., 1., 1.], frameon=False, xticks=[], yticks=[]) # Remove the white edge
S = librosa.feature.melspectrogram(y=speech, sr=sr)
librosa.display.specshow(librosa.power_to_db(S, ref=np.max))
pylab.savefig(save_path, bbox_inches=None, pad_inches=0)
pylab.close()

save_path1 = 'test_lin.png'
pylab.axes([0., 0., 1., 1.], frameon=False, xticks=[], yticks=[]) # Remove the white edge
D = librosa.amplitude_to_db(lin, ref=np.max)
librosa.display.specshow(D, y_axis='linear')
pylab.savefig(save_path1, bbox_inches=None, pad_inches=0)
pylab.close()
```

初次导入`A2_2.wav`音频，通过该代码生成`test.wav`音频，对比发现两音频的发声内容相同，但音色存在较大差异。为测试是哪部分出现问题，将`test.wav`作为输入生成`test1.wav`，发现test与test1的音色较为相近且test1的音量较小，初步判断导致音色存在差异的原因是librosa的griffin_lim算法可能存在一定的实现偏差。且该代码的mel与mag提取存在一定的特征流失的情况。

对应mel与mag谱图如下：

![mel](https://p.qlogo.cn/qqmail_head/tFvCianWnUl4FlklZ5cbcsR599vrZLo2I0wo0icxKIMwyBdXg24ObIyb1ia5ln5MyJLWMNKuw3qKko/0)

![mag](https://p.qlogo.cn/qqmail_head/tFvCianWnUl4FlklZ5cbcsR599vrZLo2I0wo0icxKIMwxn3BSyjEbGkaEEeIicvkl29d76TrKoCttA/0)

#### 2.关键代码自实现该过程

代码：引自网络

```python
import scipy.signal as signal
import librosa
import torch
import numpy as np
import copy

sr = 22050 # Sample rate.
n_fft = 2048 # fft points (samples)
frame_shift = 0.0125 # seconds
frame_length = 0.05 # seconds
hop_length = int(sr*frame_shift) # samples.
win_length = int(sr*frame_length) # samples.
n_mels = 80 # Number of Mel banks to generate
power = 1.2 # Exponent for amplifying the predicted magnitude
n_iter = 100 # Number of inversion iterations
preemphasis = .97 # or None
max_db = 100
ref_db = 20
top_db = 15


def get_spectrograms(fpath):
    '''Returns normalized log(melspectrogram) and log(magnitude) from `sound_file`.
    Args:
      sound_file: A string. The full path of a sound file.

    Returns:
      mel: A 2d array of shape (T, n_mels) <- Transposed
      mag: A 2d array of shape (T, 1+n_fft/2) <- Transposed
 '''
    # Loading sound file
    y, sr = librosa.load(fpath, sr=22050)

    # Trimming
    y, _ = librosa.effects.trim(y, top_db=top_db)

    # Preemphasis
    y = np.append(y[0], y[1:] - preemphasis * y[:-1])

    # stft
    linear = librosa.stft(y=y,
                          n_fft=n_fft,
                          hop_length=hop_length,
                          win_length=win_length)

    # magnitude spectrogram
    mag = np.abs(linear)  # (1+n_fft//2, T)

    # mel spectrogram
    mel_basis = librosa.filters.mel(sr, n_fft, n_mels)  # (n_mels, 1+n_fft//2)
    mel = np.dot(mel_basis, mag)  # (n_mels, t)

    # to decibel
    mel = 20 * np.log10(np.maximum(1e-5, mel))
    mag = 20 * np.log10(np.maximum(1e-5, mag))

    # normalize
    mel = np.clip((mel - ref_db + max_db) / max_db, 1e-8, 1)
    mag = np.clip((mag - ref_db + max_db) / max_db, 1e-8, 1)

    # Transpose
    mel = mel.T.astype(np.float32)  # (T, n_mels)
    mag = mag.T.astype(np.float32)  # (T, 1+n_fft//2)

    return mel, mag

def melspectrogram2wav(mel):
    '''# Generate wave file from spectrogram'''
    # transpose
    mel = mel.T

    # de-noramlize
    mel = (np.clip(mel, 0, 1) * max_db) - max_db + ref_db

    # to amplitude
    mel = np.power(10.0, mel * 0.05)
    m = _mel_to_linear_matrix(sr, n_fft, n_mels)
    mag = np.dot(m, mel)

    # wav reconstruction
    wav = griffin_lim(mag)

    # de-preemphasis
    wav = signal.lfilter([1], [1, -preemphasis], wav)

    # trim
    wav, _ = librosa.effects.trim(wav)

    return wav.astype(np.float32)


def spectrogram2wav(mag):
    '''# Generate wave file from spectrogram'''
    # transpose
    mag = mag.T

    # de-noramlize
    mag = (np.clip(mag, 0, 1) * max_db) - max_db + ref_db

    # to amplitude
    mag = np.power(10.0, mag * 0.05)

    # wav reconstruction
    wav = griffin_lim(mag)

    # de-preemphasis
    wav = signal.lfilter([1], [1, -preemphasis], wav)

    # c
    wav, _ = librosa.effects.trim(wav)

    return wav.astype(np.float32)



def _mel_to_linear_matrix(sr, n_fft, n_mels):
    m = librosa.filters.mel(sr, n_fft, n_mels)
    m_t = np.transpose(m)
    p = np.matmul(m, m_t)
    d = [1.0 / x if np.abs(x) > 1.0e-8 else x for x in np.sum(p, axis=0)]
    return np.matmul(m_t, np.diag(d))


def griffin_lim(spectrogram):
    '''Applies Griffin-Lim's raw.
    '''
    X_best = copy.deepcopy(spectrogram)
    for i in range(n_iter):
        X_t = invert_spectrogram(X_best)
        est = librosa.stft(X_t, n_fft, hop_length, win_length=win_length)
        phase = est / np.maximum(1e-8, np.abs(est))
        X_best = spectrogram * phase
    X_t = invert_spectrogram(X_best)
    y = np.real(X_t)

    return y


def invert_spectrogram(spectrogram):
    '''
    spectrogram: [f, t]
    '''
    return librosa.istft(spectrogram, hop_length, win_length=win_length, window="hann")

def plot_spectrogram_to_numpy(spectrogram):
    fig, ax = plt.subplots(figsize=(12, 3))
    im = ax.imshow(spectrogram, aspect="auto", origin="lower",
                   interpolation='none')
    plt.colorbar(im, ax=ax)
    plt.xlabel("Frames")
    plt.ylabel("Channels")
    plt.tight_layout()

    fig.canvas.draw()
    data = save_figure_to_numpy(fig)
    plt.close()
    return data

def save_figure_to_numpy(fig):
    # save it to a numpy array.
    data = np.fromstring(fig.canvas.tostring_rgb(), dtype=np.uint8, sep='')
    data = data.reshape(fig.canvas.get_width_height()[::-1] + (3,))
    return data

if __name__ == '__main__':

    aa = get_spectrograms('../../speech/A2_2.wav')
    import matplotlib.pyplot as plt

    plt.figure()
    plt.subplot(2, 1, 2)
    plt.imshow(plot_spectrogram_to_numpy(aa[1].T))
    plt.subplot(2, 1, 1)
    plt.imshow(plot_spectrogram_to_numpy(aa[0].T))
    plt.show()


    wav = melspectrogram2wav(aa[0])
    librosa.output.write_wav("mel2wav.wav", wav, sr)
    #wav = spectrogram2wav(aa[1])
    #librosa.output.write_wav("mag2wav.wav", wav, sr)
```

该代码能够较好的将提取到的mel谱图与mag转化成真实音频。但是mel转音频与mag转音频间存在一定差异，mel转换效果较弱于mag，即mel转换得到的音频人耳听起来较为模糊，实现效果较差。

mag与mel在该代码中的显示如下：

![](https://p.qlogo.cn/qqmail_head/tFvCianWnUl4FlklZ5cbcsR599vrZLo2I0wo0icxKIMwzexawG42X32AVibziaiaFfeXvRJS3L85Njw8/0)