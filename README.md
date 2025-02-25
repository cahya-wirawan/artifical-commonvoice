# Artificial Common Voice
Common Voice is a crowdsourcing project started by Mozilla to create a free database for speech recognition software.
It aims to provide diverse voice samples from different languages. Its dataset contains more than 1600 hours of
validated English voices, however many other languages are still under presented. Low resource languages are a challenge
for the training of speech recognition model.

To mitigate this issue a little, I create Artificial Common Voice. It generates Common Voices artificially using 
different Speech Synthesizer services such as [Google Text To Speech](https://cloud.google.com/text-to-speech) 
or [Azure Text To Speech](https://azure.microsoft.com/en-us/services/cognitive-services/text-to-speech/) 
(Currently, only Google Text To Speech is supported). It reads the tab-separated value (tsv) file provided by 
Common Voice, which contains client-id, sentence, the sound file path, and other information. It sends the sentence 
one by one to the Speech Synthesizer service like Google TTS and stores the retrieved sound files.

## The Cost of Google Text To Speech
According to the [Google Text To Speech Pricing](https://cloud.google.com/text-to-speech/pricing), the first 4M 
characters for the standard voice types, and the first 1M characters for Wavenet voice types are free per month.
<img src="https://github.com/cahya-wirawan/artifical-commonvoice/blob/main/images/Google%20TTS%20Pricing.png" 
alt="Google TTS Pricing" style="width: 300px;"/>

## Prerequisite

Before you start with this script, first you have to enable the Google Cloud Text-to-Speech API, create service account
and its key, and save it locally. You will use this service account key for the environment variable 
GOOGLE_APPLICATION_CREDENTIALS. Please follow this 
[Google Cloud Text-to-Speech API Setup](https://cloud.google.com/text-to-speech/docs/quickstart-client-libraries)
to do all above.

Then you have to install the python module google-cloud-texttospeech
```
% pip install google-cloud-texttospeech
```

## Usage

### List of commands
```
% export GOOGLE_APPLICATION_CREDENTIALS="path to the google authentication credentials"
% python commonvoice.py
usage: commonvoice.py [-h] [-c COMMONVOICE_FILE] [-o OUTPUT_DIR] [-l]
                       [-v VOICE_TYPES [VOICE_TYPES ...]] [--random_pitch]
                       [--random_pitch_minmax RANDOM_PITCH_MINMAX]
                       [--random_speed]
                       [--random_speed_minmax RANDOM_SPEED_MINMAX] [-q] [-d]
                       [-s] [-t SLEEP_TIME]

optional arguments:
  -h, --help            show this help message and exit
  -c COMMONVOICE_FILE, --commonvoice_file COMMONVOICE_FILE
                        Common Voice file, a tab separates value file contains
                        client_id, path, sentence, etc..
  -o OUTPUT_DIR, --output_dir OUTPUT_DIR
                        Output directory where the sound files will be stored
  -l, --list_voice_types
                        List supported voice types
  -v VOICE_TYPES [VOICE_TYPES ...], --voice_types VOICE_TYPES [VOICE_TYPES ...]
                        List of voice types such as id-ID-Standard-A or id-ID-
                        Wavenet-B
  --random_pitch        Enable random pitch between -random_pitch_minmax to
                        random_pitch_minmax
  --random_pitch_minmax RANDOM_PITCH_MINMAX
                        The value for random pitch
  --random_speed        Enable random speed between (1.0-random_speed_minmax)
                        to (1+random_pitch_minmax)
  --random_speed_minmax RANDOM_SPEED_MINMAX
                        The value for random speed
  -q, --quite           Disable info about a successful sound file creation
  -d, --debug           Enable debug messages
  -s, --sleep           Enable sleep in second between request
  -t SLEEP_TIME, --sleep_time SLEEP_TIME
                        Sleep time in second between request
```
### List all supported voice types for all languages
```
% python commonvoice.py -l
...
voices {
  language_codes: "id-ID"
  name: "id-ID-Wavenet-D"
  ssml_gender: FEMALE
  natural_sample_rate_hertz: 24000
}
voices {
  language_codes: "id-ID"
  name: "id-ID-Wavenet-A"
  ssml_gender: FEMALE
  natural_sample_rate_hertz: 24000
}
voices {
  language_codes: "id-ID"
  name: "id-ID-Wavenet-B"
  ssml_gender: MALE
  natural_sample_rate_hertz: 24000
}
...
```

### Generate sound files 
Get the tsv file from Common Voice dataset. If you have already download it using Huggingface datasets, 
you can find these files for example using folllowing command:
```
% find ~/.cache/huggingface -name "*.tsv"
/home/cahya/.cache/huggingface/datasets/downloads/extracted/fd8a16a97efd77adba3c26c54d0cfae6c9d9494c1017f8070f3f79db72c4b57c/cv-corpus-6.1-2020-12-11/id/invalidated.tsv
/home/cahya/.cache/huggingface/datasets/downloads/extracted/fd8a16a97efd77adba3c26c54d0cfae6c9d9494c1017f8070f3f79db72c4b57c/cv-corpus-6.1-2020-12-11/id/other.tsv
/home/cahya/.cache/huggingface/datasets/downloads/extracted/fd8a16a97efd77adba3c26c54d0cfae6c9d9494c1017f8070f3f79db72c4b57c/cv-corpus-6.1-2020-12-11/id/reported.tsv
/home/cahya/.cache/huggingface/datasets/downloads/extracted/fd8a16a97efd77adba3c26c54d0cfae6c9d9494c1017f8070f3f79db72c4b57c/cv-corpus-6.1-2020-12-11/id/test.tsv
/home/cahya/.cache/huggingface/datasets/downloads/extracted/fd8a16a97efd77adba3c26c54d0cfae6c9d9494c1017f8070f3f79db72c4b57c/cv-corpus-6.1-2020-12-11/id/train.tsv
/home/cahya/.cache/huggingface/datasets/downloads/extracted/fd8a16a97efd77adba3c26c54d0cfae6c9d9494c1017f8070f3f79db72c4b57c/cv-corpus-6.1-2020-12-11/id/validated.tsv
```
Now you can use this tsv file to generate sound files. Following is an example to generate sound files for Indonesian 
Common Voice with two voice types id-ID-Standard-A id-ID-Wavenet-B and store it in output directory:
```
% python commonvoice.py -c validated.tsv -v id-ID-Standard-A id-ID-Wavenet-B -o "./output"
```
You can also create your own tsv files. I included here the file voice-id.tsv as an example
```
% cat voice-id.tsv 
path    sentence
test01.mp3      Kleopatra adalah penguasa aktif terakhir Kerajaan Wangsa Ptolemaios di tanah Mesir.
test02.mp3      Serealia adalah biji-bijian yang dihasilkan dari tanaman famili Poaceae.
%
% python commonvoice.py -c voice-id.tsv -v id-ID-Standard-A id-ID-Wavenet-B -o "./test"
```
This command will create following file structure:
```
test
├── id-ID-Standard-A
│   ├── test01.mp3
│   └── test02.mp3
└── id-ID-Wavenet-B
    ├── test01.mp3
    └── test02.mp3

```
Please keep in mind that the generated sound files have a 24000Hz sampling rate, where Common Voice sound files have 
48000Hz sampling rate. You might have to change the code for resampling the sound file to a 16000Hz sampling rate.

In case you want to generate a bunch of sound files from the Common Voice, Google TTS has a limitation of
300 requests per minute. I didn't reach this limit when I generated sound files with Wavenet voice types, but it is 
not the case with the standard voice types. Its generation is swift, so I reached the limit. To avoid it, the script
provides functionality to sleep 100ms after every request. This can be done using the argument -s for enabling the 
sleep time and -t for the sleep time (default value is 100ms).
