# Recognize speech with an IoT device

![A sketchnote overview of this lesson](../../../sketchnotes/lesson-21.jpg)

> Sketchnote by [Nitya Narasimhan](https://github.com/nitya). Click the image for a larger version.

This video gives an overview of the Azure speech service, a topic that will be covered in this lesson:

[![How to get started using your Cognitive Services Speech resource from the Microsoft Azure YouTube channel](https://img.youtube.com/vi/iW0Fw0l3mrA/0.jpg)](https://www.youtube.com/watch?v=iW0Fw0l3mrA)

> 🎥 Click the image above to watch a video

## Pre-lecture quiz

[Pre-lecture quiz](https://black-meadow-040d15503.1.azurestaticapps.net/quiz/41)

## Introduction

'Alexa, set a 12 minute timer'

'Alexa, timer status'

'Alexa, set a 8 minute timer called steam broccoli'

Smart devices are becoming more and more pervasive. Not just as smart speakers like HomePods, Echos and Google Homes, but embedded in our phones, watches, and even light fittings and thermostats.

> 💁 I have at least 19 devices in my home that have voice assistants, and that's just the ones I know about!

Voice control increases accessibility by allowing folks with limited movement to interact with devices. Whether it is a permanent disability such as being born without arms, to temporary disabilities such as broken arms, or having your hands full of shopping or young children, being able to control our houses from our voice instead of our hands opens up a world of access. Shouting 'Hey Siri, close my garage door' whilst dealing with a baby change and an unruly toddler can be a small but effective improvement on life.

One of the more popular uses for voice assistants is setting timers, especially kitchen timers. Being able to set multiple timers with just your voice is a great help in the kitchen - no need to stop kneading dough, stirring soup, or clean dumpling filling off your hands to use a physical timer.

In this lesson you will learn about building voice recognition into IoT devices. You'll learn about microphones as sensors, how to capture audio from a microphone attached to an IoT device, and how to use AI to convert what is heard into text. Throughout the rest of this project you will build a smart kitchen timer, able to set timers using your voice with multiple languages.

In this lesson we'll cover:

- [Microphones](#microphones)
- [Capture audio from your IoT device](#capture-audio-from-your-iot-device)
- [Speech to text](#speech-to-text)
- [Convert speech to text](#convert-speech-to-text)

## Microphones

Microphones are analog sensors that convert sound waves into electrical signals. Vibrations in air cause components in the microphone to move tiny amounts, and these cause tiny changes in electrical signals. These changes are then amplified to generate an electrical output.

### Microphone types

Microphones come in a variety of types:

- Dynamic - Dynamic microphones have magnet attached to a moving diaphragm that moves in a coil of wire creating an electrical current. This is the opposite of most loudspeakers, that use an electrical current to move a magnet in a coil of wire, moving a diaphragm to create sound. This means speakers can be used a dynamic microphones, and dynamic microphones can be used as speakers. In devices such as intercoms where a user is either listening or speaking, but not both, one device can act as both a speaker and a microphone.

  Dynamic microphones don't need power to work, the electrical signal is created entirely from the microphone.

  ![Patti Smith singing into a Shure SM58 (dynamic cardioid type) microphone](../../../images/dynamic-mic.jpg)

- Ribbon - Ribbon microphones are similar to dynamic microphones, except they have a metal ribbon instead of a diaphragm. This ribbon moves in a magnetic field generating an electrical current. Like dynamic microphones, ribbon microphones don't need power to work.

  ![Edmund Lowe, American actor, standing at radio microphone (labeled for (NBC) Blue Network), holding script, 1942](../../../images/ribbon-mic.jpg)

- Condenser - Condenser microphones have a thin metal diaphragm and a fixed metal backplate. Electricity is applied to both of these and as the diaphragm vibrates the static charge between the plates changes generating a signal. Condenser microphones need power to work - called _Phantom power_.

  ![C451B small-diaphragm condenser microphone by AKG Acoustics](../../../images/condenser-mic.jpg)

- MEMS - Microelectromechanical systems microphones, or MEMS, are microphones on a chip. They have a pressure sensitive diaphragm etched onto a silicon chip, and work similar to a condenser microphone. These microphones can be tiny, and integrated into circuitry.

  ![A MEMS microphone on a circuit board](../../../images/mems-microphone.png)

  In the image above, the chip labelled **LEFT** is a MEMS microphone, with a tiny diaphragm less than a millimeter wide.

✅ Do some research: What microphones do you have around you - either in your computer, your phone, your headset or in other devices. What type of microphones are they?

### Digital audio

Audio is an analog signal carrying very fine-grained information. To convert this signal to digital, the audio needs to be sampled many thousands of times a second.

> 🎓 Sampling is converting the audio signal into a digital value that represents the signal at that point in time.

![A line chart showing a signal, with discrete points at fixed intervals](../../../images/sampling.png)

Digital audio is sampled using Pulse Code Modulation, or PCM. PCM involves reading the voltage of the signal, and selecting the closest discrete value to that voltage using a defined size.

> 💁 You can think of PCM as the sensor version of pulse width modulation, or PWM (PWM was covered back in [lesson 3 of the getting started project](../../../1-getting-started/lessons/3-sensors-and-actuators/README.md#pulse-width-modulation)). PCM involves converting an analog signal to digital, PWM involves converting a digital signal to analog.

For example most streaming music services offer 16-bit or 24-bit audio. This means they convert the voltage into a value that fits into a 16-bit integer, or 24-bit integer. 16-bit audio fits the value into a number ranging from -32,768 to 32,767, 24-bit is in the range −8,388,608 to 8,388,607. The more bits, the closer the sample is to what our ears actually hear.

> 💁 You may have hard of 8-bit audio, often referred to as LoFi. This is audio sampled using only 8-bits, so -128 to 127. The first computer audio was limited to 8 bits due to hardware limitations, so this is often seen in retro gaming.

These samples are taken many thousands of times per second, using well-defined sample rates measured in KHz (thousands of readings per second). Streaming music services use 48KHz for most audio, but some 'lossless' audio uses up to 96KHz or even 192KHz. The higher the sample rate, the closer to the original the audio will be, up to a point. There is debate whether humans can tell the difference above 48KHz.

✅ Do some research: If you use a streaming music service, what sample rate and size does it use? If you use CDs, what is the sample rate and size of CD audio?

There are a number of different formats for audio data. You've probably heard of mp3 files - audio data that is compressed to make it smaller without losing any quality. Uncompressed audio is often stored as a WAV file - this is a file with 44 bytes of header information, followed by raw audio data. The header contains information such as the sample rate (for example 16000 for 16KHz) and sample size (16 for 16-bit), and the number of channels. After the header, the WAV file contains the raw audio data.

> 🎓 Channels refers to how many different audio streams make up the audio. For example, for stereo audio with left and right, there would be 2 channels. For 7.1 surround sound for a home theater system this would be 8.

### Audio data size

Audio data is relatively large. For example, capturing uncompressed 16-bit audio at 16KHz (a good enough rate for use with speech to text model), takes 32KB of data for each second of audio:

- 16-bit means 2 bytes per sample (1 byte is 8 bits).
- 16KHz is 16,000 samples per second.
- 16,000 x 2 bytes = 32,000 bytes per second.

This sounds like a small amount of data, but if you are using a microcontroller with limited memory, this can be a lot. For example, the Wio Terminal has 192KB of memory, and that needs to store program code and variables. Even if your program code was tiny, you couldn't capture more than 5 seconds of audio.

Microcontrollers can access additional storage, such as SD cards or flash memory. When building an IoT device that captures audio you will need to ensure not only you have additional storage, but your code writes the audio captured from your microphone directly to that storage, and when sending it to the cloud, you stream from storage to the web request. That way you can avoid running out of memory by trying to hold the entire block of audio data in memory at once.

## Capture audio from your IoT device

Your IoT device can be connected to a microphone to capture audio, ready for conversion to text. It can also be connected to speakers to output audio. In later lessons this will be used to give audio feedback, but it is useful to set up speakers now to test the microphone.

### Task - configure your microphone and speakers

Work through the relevant guide to configure the microphone and speakers for your IoT device:

- [Arduino - Wio Terminal](wio-terminal-microphone.md)
- [Single-board computer - Raspberry Pi](pi-microphone.md)
- [Single-board computer - Virtual device](virtual-device-microphone.md)

### Task - capture audio

Work through the relevant guide to capture audio on your IoT device:

- [Arduino - Wio Terminal](wio-terminal-audio.md)
- [Single-board computer - Raspberry Pi](pi-audio.md)
- [Single-board computer - Virtual device](virtual-device-audio.md)

## 음성을 텍스트로 변환

음성을 텍스트로 변환하거나 음성 인식은 AI를 사용하여 오디오 신호의 단어를 텍스트로 변환하는 것을 포함합니다.

### 음성 인식 모델

음성을 텍스트로 변환하기 위해, 오디오 신호의 샘플은 함께 그룹화되어 Recurrent Neural network (RNN) 기반의 기계 학습 모델에 제공됩니다. 이는 이전 데이터를 사용하여 수신 데이터에 대한 결정을 내릴 수 있는 일종의 기계 학습 모델입니다. 예를 들어, RNN은 오디오 샘플의 한 블록을 'Hel'로 감지할 수 있고, 다른 블록을 수신하면 이를 'lo' 라고 생각할 수 있습니다. 이를 이전 블록과 결합하여 'Hello' 가 올바른 단어임을 찾고, 결과 값으로 선택합니다.

ML 모델은 항상 동일한 크기의 데이터를 받아들입니다. 이전 단원에서 구축한 이미지 분류기는 이미지의 크기를 고정된 크기로 조정하고 처리합니다. 이는 음성 모델도 마찬가지로, 오디오 청크 사이즈를 조정해야 합니다. 음성 모델은 'Hi'과 'Highway', 또는 'flock'과 'floccinaucinihilipilification' 을 구별할 수 있도록, 여러 예측의 출력을 결합하여 답을 얻을 수 있어야 합니다.

또한 음성 모델은 문단을 이해할 수 있을 정도로 발전했으며, 더 많은 소리를 처리해 감지한 언어를 수정할 수 있습니다. 예를 들어, "I went to the shops to get two bananas and an apple too" 라고 말한다면, 동음의이어 to, two, too 를 사용할 수 있습니다. 음성 모델은 문단을 이해하고 절절한 철자를 사용할 수 있습니다.

> 💁 일부 음성 서비스들은 공장과 같은 시끄러운 환경이나, 화학 이름과 같이 특정 산업에서 사용되는 단어들을 맞춤화하여 더 잘 작동하도록 할 수 있습니다. 이러한 맞춤화는 주어진 샘플 오디오와 사본, 전이 학습으로 훈련됩니다. 이전 시간에 몇 개의 이미지를 사용하여 이미지 분류기를 휸련한 것과 동일한 방법입니다.

### 개인 정보

소비자 IoT 장치에서 음성을 텍스트로 변환할때, 개인 정보는 매우 중요합니다. 이러한 장치는 지속적으로 오디오를 듣습니다. 소비자는 말하는 모든 내용이 클라우드로 전송되어 텍스트로 변환되는 것을 원하지 않을 수 있습니다. 이것은 많은 인터넷 대역폭을 사용할 뿐만 아니라, 개인 정보에 중요한 영향을 미칩니다. 특히 일부 스마트 장치 제조업체가 [모델을 개선하는데 도움이 되도록, 생성된 텍스트에 대해 사람이 검증](https://www.theverge.com/2019/4/10/18305378/amazon-alexa-ai-voice-assistant-annotation-listen-private-recordings)할 오디오를 선택할 때 큰 영향을 미칩니다.

당신은 스마트 장치가 집이나, 사적인 회의, 또는 친밀한 사이에서의 대화가 아니라, 장치를 이용할 때만 클라우드에 오디오를 전송해 처리해주기를 원할 것입니다. 이를 위해, 대부분의 스마트 장치가 이용하는 방식은 호출 명령어입니다. "Alexa", "Hey Siri", 또는 "OK Google"과 같은 _호출 명령어_ 를 사용해 장치를 "깨워" 사용자가 말하는 것을 듣게 합니다. 장치는 음성 중단을 감지하여 대화가 마쳤음을 알아챕니다.

> 🎓 호출 명령어는 _키워드 발견_ 또는 _키워드 인식_ 이라고도 합니다.

이러한 호출 명령어는 클라우드가 아닌 장치에서 감지됩니다. 이러한 스마트 장치들은 작은 AI 모델이 있으며, 이는 장치를 깨우는 작업을 수행합니다. 호출 명령어가 감지되면 인식을 위해 오디오를 클라우드로 스트리밍하기 시작합니다. 이 모델은 매우 전문화되어 있으며 호출 명령어를 듣기만 하면 됩니다.

> 💁 일부 기술 회사는 장치에 더 많은 개인 정보를 추가하고 장치에서 음성을 텍스트로 변환하는 작업을 수행하고 있습니다. Apple은 장치에서 음성을 텍스트로 변환하는 기능을 지원하고, 클라우드를 사용하지 않고 많은 요쳥을 처리할 수 있는 기능에 대한 2021년 iOS 와 macOS 업데이트 일부를 발표했습니다. 이는 기기에 ML 모델을 실행할 수 있는 강력한 프로세서가 있기에 가능했습니다.

✅ 클라우드로 오디오를 전송하고 저장하는 것의 개인 정보 보호 및 윤리적 의미는 무엇이라고 생각합니까? 오디오를 저장해야 한다면 어떻게 해야 합니까? 법 집행을 위해 녹음을 사용하는 것이 사생활 침해에 대한 좋은 절충안이라고 생각힙니까?

호출 명령어 감지는 주로 TinyMl이라는 기술을 사용합니다. 이는 ML 모델을 마이크로컨트롤러에서 실행할 수 있도록 변환한 것입니다. 이 모델은 크기가 매우 작고, 실행하는데 매우 적은 전력을 소모합니다.

호출 명령어 모델을 사용과 복잡한 훈련을 피하기 위해, 이 단원에서 구축하는 스마트 타이머는 버튼을 사용하여 음성 인식을 시작합니다.

> 💁 Wio Terminal 또는 Raspberry Pi 에서 wake world 감지 모델을 생성하려는 경우, 다음 문서 [Edge Impulse로 음성에 대한 응답](https://docs.edgeimpulse.com/docs/responding-to-your-voice)를 확인하십시오. 컴퓨터를 사용하여 작업을 수행하려는 경우, [Microsoft의 사용자 지정 키워드와 시작하기 문서](https://docs.microsoft.com/azure/cognitive-services/speech-service/keyword-recognition-overview?WT.mc_id=academic-17441-jabenn)에서 시도할 수 있습니다.

## 음성을 텍스트로 변환

![Speech services logo](../../../../images/azure-speech-logo.png)

이전 프로젝트의 이미지 분류와 마찬가지로, 음성을 오디오 파일로 받아 텍스트로 변환할 수 있는 미리 구축된 AI 서비스가 있습니다. 그러한 서비스가 인지 서비스의 일부인 음성 서비스라면, 당신의 앱에서 사용할 수 있는 미리 만들어진 인공지능 서비스입니다.

### 작업 - 음성 AI 리소스 구성하기

1. `smart-timer`라는 이 프로젝트에 대한 리소스 그룹을 만듭니다.

2. 다음 명령을 사용해 자유 연설 리소스를 만듭니다:

   ```sh
   az cognitiveservices account create --name smart-timer \
                                       --resource-group smart-timer \
                                       --kind SpeechServices \
                                       --sku F0 \
                                       --yes \
                                       --location <location>
   ```

   `<location>` 를 리소스 그룹을 생성할 때 사용한 위치로 바꿉니다.

3. 코드에서 음성 리소스에 접근하려면 API 키가 필요합니다. 다음 명령을 실행하여 key를 가져옵니다:

   ```sh
   az cognitiveservices account keys list --name smart-timer \
                                          --resource-group smart-timer \
                                          --output table
   ```

   키 중 하나를 복사하십시오.

### 작업 - 음성을 텍스트로 변환하기

관련 가이드를 통해 IoT 장치에서 음성을 텍스트로 변환합니다:

- [Arduino - Wio Terminal](wio-terminal-speech-to-text.md)
- [싱글 보드 컴퓨터 - Raspberry Pi](pi-speech-to-text.md)
- [싱글 보드 컴퓨터 - 가상 장치](virtual-device-speech-to-text.md)

---

## 🚀 Challenge

Speech recognition has been around for a long time, and is continuously improving. Research the current capabilities and compare how these have evolved over time, including how accurate machine transcriptions are compared to human.

What do you think the future holds for speech recognition?

## Post-lecture quiz

[Post-lecture quiz](https://black-meadow-040d15503.1.azurestaticapps.net/quiz/42)

## Review & Self Study

- Read about the different microphone types and how they work on the [what's the difference between dynamic and condenser microphones article on Musician's HQ](https://musicianshq.com/whats-the-difference-between-dynamic-and-condenser-microphones/).
- Read more on the Cognitive Services speech service on the [speech service documentation on Microsoft Docs](https://docs.microsoft.com/azure/cognitive-services/speech-service/?WT.mc_id=academic-17441-jabenn)
- Read about keyword spotting on the [keyword recognition documentation on Microsoft Docs](https://docs.microsoft.com/azure/cognitive-services/speech-service/keyword-recognition-overview?WT.mc_id=academic-17441-jabenn)

## Assignment

[](assignment.md)
