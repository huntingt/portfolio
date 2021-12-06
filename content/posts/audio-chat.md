+++
title = "Audio Chat"
date = "2021-12-05T11:11:39-05:00"
draft = false
+++

{{<image src="/demo.png" position="center"
    style="border-radius: 8px; width:50%;">}}

I made this simple audio chat software to play with signal modulation and
encoding. The signal is encoded using 4-QAM or 16-QAM and uses matched SRRC
filters. The hardest part of the system was
the preamble detection used to start and determine the corrections necessary
for a signal to be received. The software is also complicated by the fact
that it is designed to operate in a chat system and thus does the signal
detection and demodulation in real time.

# Signal Detection

## First Attempt

When I was first designing the signal detection, I stayed away from correlating
the expected preamble samples against the signal and instead followed a state
machine based approach that requires less computation. I did this by holding
the modulating a `1+0.j` in the QI plane for `N` symbols. I would then transmit
`-1+0.j` followed by `1+0.j` before finally starting the transmission. I could
first sample the center 50% of the initial preamble and average that to correct
for the phase and amplitude. The short reversal to `-1+0.j` then provided a peak that I
could fit against a quadratic to synchronize the time.

I could then accept or reject the corrections based
off of minimum amplitude and maximum amplitude variation requirements. This
inital approach, however, was very tempermental because the time
synchronization portion of the signal was very short and extremely sensitive to
noise.

## Second Attempt

I improved this initial approach by instead alternating between peaks of
amplitude 1 at phase offsets of 90 degrees and signals of zero. Since we don't
know the signal phase offset when detecting the preamble, the preamble is
really encoded in the phase differences.

The detection state machine works by recording the most recent `N` peaks and the
times that they occured at, where `N` is the number of peaks in the preamble.
Peaks are detected by entering a detection state once the signal exceeds 80% of
the maxium previously detected peak. Once the signal falls bellow 60% of the
current peak maximum, then the peak is recorded as the maxium value in the
interval.

This system does have the problem that spurious noise in the signal band may
register as an unusually large peak that raises the 80% detection threshold
above any legitimant preamble signals, causing the system to lockup. Since we
know the length of the preamble, however, the system simply discards all peaks
that are too old to detect.

Once a sufficient number of peaks have been detected, the detection times can
be fit to determine the timing offset, and the amplitudes can be multiplied by
the complex conjugate of the preamble to correct the amplitude and phase
offsets. If the preamble has acceptibly accurate corrections, then it will be
accepted and the signal detection will start.

This approach has the nice side effect that the minimum detection threshold can be
significanly lower than in my first approach since this method can
effictively be interrupted even if it has spurious peaks of smaller amplitude.

## Performance

1000 random bits modulated with 16-QAM were sent over a AWGN channel with
varying noise amplitudes and I measured the SNR of the signal and the MER.

| Signal SNR (dB) | Constellation MER (dB) |
| -------- | -------- |
| 51.7     | 28.4     |
| 23.3     | 22.5     |
| 17.6     | 17.0     |
| 15.9     | 15.8     |
| < ~15    | no detection |

The preamble detector seems to have a minimum noise floor, but otherwise
performs fairly well under more challenging situations. Bellow an SNR of about
15 dB, the preamble detection stops working even though the BER is typically <
0.01.

An example constellation is shown below:

{{<image src="/16qam.png" position="center"
    style="border-radius: 8px; width:50%;"
    caption="Hi there">}}

# Problems

Currently, the audio chat app works well when sending and receiving data
between a computer and itself. When trying to communicate between computers,
communication reliability can be rather hit or miss, depending on the location
and the computers. I suspect that the speaker and microphone's frequency
response are non trivial, and echos along with any post processing in the
OS are making this very difficult. Ultimantly, I need to do a good job
characterizing the channel in different configurations to know for sure.

## Decoding

I would also like to try incorporating soft decoding by using something like a
viterbi decoder. I haven't been able to find literature, however, that shows
how to do this in a fairly general method that would allow puncturing to fine
tune the code rate. The problem is that the codes are in terms of bits while
the modem is communicating in symbols in some QAM constellation. So to assign a
branch metric can be rather hard since there are multiple symbols that may have
a bit, and not all bits are in a symbol if code puncturing is used to increase
the code rate.

# Links

- [Source Code](https://github.com/huntingt/audio-chat)
