# Digital Audio Fundamentals

## Signal Paths

Audio has three mediums: physical (pressure waves), analog (voltage signal), and digital (discrete data).

**Audio transformation** is the conversion of one medium to another, like so:

Physical <--> Analog <--> Digital

Note that the physical domain and digital domain require the analog domain as an intermediary. The act of changing domains will *always* result in some sort of unavoidable data loss, and therefore should generally be minimized as much as possible.

Consider the case where a vocalist sings into a microphone, that signal is sent to a pre-amp where the signal is boosted, and then that signal gets sent to a speaker. In that instance there are two stages of conversion: one from the physical to analog, and then analog to physical.

Consider another example where the vocalist is recording herself. Her mic is hooked up to an audio interface (which houses a pre-amp, analog to digital converter, and a digital to audio converter). Her audio interface is hooked up to a computer running a DAW that is recording the audio. When the vocalist wants to hear herself back, the data is sent back from her computer to the audio interface and then is transferred to her studio monitors for playback. In this case there are 4 stages of conversion: physical to analog, analog to digital, digital to analog, and analog to physical.

**Audio processing** is the modification of audio *within the same domain*. In many cases this processing is lossless and reversible.

## Sampling Theorem

Continuous-to-discrete signal conversion is governed by the Nyquist Shannon Sampling Theorem:

"Any **band limited** continuous-time signal can be accurately converted to and from digital signals when **sampled at a rate, at least twice as high as the highest frequency** component of the waveform."

This theorem states we need at least *more than* 2 samples to capture one period of a wave. Note that only capturing 2 samples per period could potentially capture the null points of the wave (each sample point would have an amplitude of zero), and therefore would insufficiently represent the wave's frequency.

Consider a 1hz frequency. Using a sample rate of 3hz, we would capture 3 samples per period. Intuitively, 3 samples per period may not seem like much, but in reality *to increase the sample rate would not yield a higher resolution*. That is, as long as you take into account the first point of the theorem: the signal must be **band limited**.

Band limiting is the limiting of a signal's frequency domain representation to zero above a certain finite frequency. You can consider this to be like a theoretical low pass filter with infinite slope.

A side note as to why the above 3-samples-per-period example works; an audio interfaces digital-to-analog converter (DAC) contains something called a Resistor Ladder Circuit. This is a circuit baked into the hardware that treats each sample point as a square wave relative to a sample's surrounding samples. There is not curvature in the wave between samples *until the band limiting process*.

The band limiting process is special and is the entire reason the example of 3 sample points per period yields the *same* resolution wave form as compared to a much higher sample rate. Band limiting a signal at a frequency of half the sample rate yields a mathematically unique solution; there can only be one band limited signal that can pass through every sample point and that is the input frequency itself. This process effectively smooths out the square wave in the Resistor Ladder Circuit to again represent the original input signal.

This is a mathematically noiseless process! However analog is never truly noiseless. But, if the audio transformations are kept to a minimum it is never noise that can be heard by human ears.

Now, because a low pass filter of infinite slope doesn't exist, there are a few things to take into account; there has to be a buffer period between the highest frequency you wish to capture and the sampling rate / 2. This buffer period is relative to the low pass filter's slope--the steeper the slope, the narrower the buffer.

Also, any frequencies caught in this buffer will cause aliasing in the form of the frequencies folding back and creating frequencies lower than their initial values. Yuck.

The main takeaway from this is that a good buffer period requires a sample rate of 2.5 times higher than the highest expected frequency (as opposed to just 2 times higher in the ideal case). So, in general, a 44.1kHz sample rate could capture frequencies up to 17,640Hz without aliasing and a 48kHz sample rate could capture frequencies up to 19,200Hz without aliasing.

## Common Audio Sample Rates

