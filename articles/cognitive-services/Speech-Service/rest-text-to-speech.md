---
title: Text-to-speech API reference (REST) - Speech service
titleSuffix: Azure Cognitive Services
description: Learn how to use the text-to-speech REST API. In this article, you'll learn about authorization options, query options, how to structure a request and receive a response.
services: cognitive-services
author: eric-urban
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/01/2021
ms.author: eur
ms.custom: references_regions
---

# Text-to-Speech REST API

The Speech service allows you to [convert text into synthesized speech](#convert-text-to-speech) and [get a list of supported voices](#get-a-list-of-voices) for a region using a set of REST APIs. Each available endpoint is associated with a region. A subscription key for the endpoint/region you plan to use is required.

The Text-to-Speech REST API supports neural Text-to-Speech voices, which support a specific language and dialect, identified by locale.

* For a complete list of voices, see [language support](language-support.md#text-to-speech).
* For information about regional availability, see [regions](regions.md#text-to-speech).

> [!IMPORTANT]
> Costs vary for prebuilt neural voices (referred as *Neural* on the pricing page) and custom neural voices (referred as *Custom Neural* on the pricing page). For more information, see [Pricing](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

Before using this API, understand:

* The Text-to-Speech REST API requires an Authorization header. This means that you need to complete a token exchange to access the service. For more information, see [Authentication](#authentication).

> [!TIP]
> See [this article](sovereign-clouds.md) for Azure Government and Azure China endpoints.

[!INCLUDE [](../../../includes/cognitive-services-speech-service-rest-auth.md)]

## Get a list of voices

The `voices/list` endpoint allows you to get a full list of voices for a specific region/endpoint.

### Regions and endpoints

| Region | Endpoint |
|--------|----------|
| Australia East | `https://australiaeast.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Brazil South | `https://brazilsouth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Canada Central | `https://canadacentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Central US | `https://centralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| East Asia | `https://eastasia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| East US | `https://eastus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| East US 2 | `https://eastus2.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| France Central | `https://francecentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| India Central | `https://centralindia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Japan East | `https://japaneast.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Korea Central | `https://koreacentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| North Central US | `https://northcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| North Europe | `https://northeurope.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| South Africa North | `https://southafricanorth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| South Central US | `https://southcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Southeast Asia | `https://southeastasia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| UK South | `https://uksouth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| West Central US | `https://westcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| West Europe | `https://westeurope.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| West US | `https://westus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| West US 2 | `https://westus2.tts.speech.microsoft.com/cognitiveservices/voices/list` |

> [!TIP]
> [Voices in preview](language-support.md#prebuilt-neural-voices-in-preview) are only available in these 3 regions: East US, West Europe and Southeast Asia.

### Request headers

This table lists required and optional headers for text-to-speech requests.

| Header | Description | Required / Optional |
|--------|-------------|---------------------|
| `Ocp-Apim-Subscription-Key` | Your Speech service subscription key. | Either this header or `Authorization` is required. |
| `Authorization` | An authorization token preceded by the word `Bearer`. For more information, see [Authentication](#authentication). | Either this header or `Ocp-Apim-Subscription-Key` is required. |



### Request body

A body isn't required for `GET` requests to this endpoint.

### Sample request

This request only requires an authorization header.

```http
GET /cognitiveservices/voices/list HTTP/1.1

Host: westus.tts.speech.microsoft.com
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
```

### Sample response

This response has been truncated to illustrate the structure of a response.

> [!NOTE]
> Voice availability varies by region/endpoint.

```json
[

    {
    "Name": "Microsoft Server Speech Text to Speech Voice (en-US, JennyNeural)",
    "DisplayName": "Jenny",
    "LocalName": "Jenny",
    "ShortName": "en-US-JennyNeural",
    "Gender": "Female",
    "Locale": "en-US",
    "StyleList": [
      "chat",
      "customerservice",
      "newscast-casual",
      "assistant",
    ],
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "Status": "GA"
  },

    ...

     {
    "Name": "Microsoft Server Speech Text to Speech Voice (en-US, JennyMultilingualNeural)",
    "ShortName": "en-US-JennyMultilingualNeural",
    "DisplayName": "Jenny Multilingual",
    "LocalName": "Jenny Multilingual",
    "Gender": "Female",
    "Locale": "en-US",
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "SecondaryLocaleList": [
        "de-DE",
        "en-AU",
        "en-CA",
        "en-GB",
        "es-ES",
        "es-MX",
        "fr-CA",
        "fr-FR",
        "it-IT",
        "ja-JP",
        "ko-KR",
        "pt-BR",
        "zh-CN"
      ],
    "Status": "Preview"
    },
    
  ...
    
    {
    "Name": "Microsoft Server Speech Text to Speech Voice (ga-IE, OrlaNeural)",
    "DisplayName": "Orla",
    "LocalName": "Orla",
    "ShortName": "ga-IE-OrlaNeural",
    "Gender": "Female",
    "Locale": "ga-IE",
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "Status": "GA"
  },

  ...

   {
    "Name": "Microsoft Server Speech Text to Speech Voice (zh-CN, YunxiNeural)",
    "DisplayName": "Yunxi",
    "LocalName": "云希",
    "ShortName": "zh-CN-YunxiNeural",
    "Gender": "Male",
    "Locale": "zh-CN",
    "StyleList": [
      "Calm",
      "Fearful",
      "Cheerful",
      "Disgruntled",
      "Serious",
      "Angry",
      "Sad",
      "Depressed",
      "Embarrassed"
    ],
    "SampleRateHertz": "24000",
    "VoiceType": "Neural",
    "Status": "GA"
  },

    ...

]
```

### HTTP status codes

The HTTP status code for each response indicates success or common errors.

| HTTP status code | Description | Possible reason |
|------------------|-------------|-----------------|
| 200 | OK | The request was successful. |
| 400 | Bad Request | A required parameter is missing, empty, or null. Or, the value passed to either a required or optional parameter is invalid. A common issue is a header that is too long. |
| 401 | Unauthorized | The request is not authorized. Check to make sure your subscription key or token is valid and in the correct region. |
| 429 | Too Many Requests | You have exceeded the quota or rate of requests allowed for your subscription. |
| 502 | Bad Gateway    | Network or server-side issue. May also indicate invalid headers. |


## Convert Text-to-Speech

The `v1` endpoint allows you to convert Text-to-Speech using [Speech Synthesis Markup Language (SSML)](speech-synthesis-markup.md).

### Regions and endpoints

These regions are supported for Text-to-Speech using the REST API. Make sure that you select the endpoint that matches your subscription region.

[!INCLUDE [](includes/cognitive-services-speech-service-endpoints-text-to-speech.md)]

### Request headers

This table lists required and optional headers for Text-to-Speech requests.

| Header | Description | Required / Optional |
|--------|-------------|---------------------|
| `Authorization` | An authorization token preceded by the word `Bearer`. For more information, see [Authentication](#authentication). | Required |
| `Content-Type` | Specifies the content type for the provided text. Accepted value: `application/ssml+xml`. | Required |
| `X-Microsoft-OutputFormat` | Specifies the audio output format. For a complete list of accepted values, see [audio outputs](#audio-outputs). | Required |
| `User-Agent` | The application name. The value provided must be less than 255 characters. | Required |

### Audio outputs

This is a list of supported audio formats that are sent in each request as the `X-Microsoft-OutputFormat` header. Each incorporates a bitrate and encoding type. The Speech service supports 24 kHz, 16 kHz, and 8 kHz audio outputs.

```output
raw-16khz-16bit-mono-pcm            riff-16khz-16bit-mono-pcm
raw-24khz-16bit-mono-pcm            riff-24khz-16bit-mono-pcm
raw-48khz-16bit-mono-pcm            riff-48khz-16bit-mono-pcm
raw-8khz-8bit-mono-mulaw            riff-8khz-8bit-mono-mulaw
raw-8khz-8bit-mono-alaw             riff-8khz-8bit-mono-alaw
audio-16khz-32kbitrate-mono-mp3     audio-16khz-64kbitrate-mono-mp3
audio-16khz-128kbitrate-mono-mp3    audio-24khz-48kbitrate-mono-mp3
audio-24khz-96kbitrate-mono-mp3     audio-24khz-160kbitrate-mono-mp3
audio-48khz-96kbitrate-mono-mp3     audio-48khz-192kbitrate-mono-mp3
raw-16khz-16bit-mono-truesilk       raw-24khz-16bit-mono-truesilk
webm-16khz-16bit-mono-opus          webm-24khz-16bit-mono-opus
ogg-16khz-16bit-mono-opus           ogg-24khz-16bit-mono-opus
ogg-48khz-16bit-mono-opus
```

> [!NOTE]
> If your selected voice and output format have different bit rates, the audio is resampled as necessary.
> ogg-24khz-16bit-mono-opus can be decoded with [opus codec](https://opus-codec.org/downloads/)

### Request body

The body of each `POST` request is sent as [Speech Synthesis Markup Language (SSML)](speech-synthesis-markup.md). SSML allows you to choose the voice and language of the synthesized speech returned by the Text-to-Speech service. For a complete list of supported voices, see [language support](language-support.md#text-to-speech).

> [!NOTE]
> If using a custom neural voice, the body of a request can be sent as plain text (ASCII or UTF-8).

### Sample request

This HTTP request uses SSML to specify the voice and language. If the body length is long, and the resulting audio exceeds 10 minutes - it is truncated to 10 minutes. In other words, the audio length cannot exceed 10 minutes.

```http
POST /cognitiveservices/v1 HTTP/1.1

X-Microsoft-OutputFormat: raw-24khz-16bit-mono-pcm
Content-Type: application/ssml+xml
Host: westus.tts.speech.microsoft.com
Content-Length: 225
Authorization: Bearer [Base64 access_token]

<speak version='1.0' xml:lang='en-US'><voice xml:lang='en-US' xml:gender='Male'
    name='en-US-ChristopherNeural'>
        Microsoft Speech Service Text-to-Speech API
</voice></speak>
```

### HTTP status codes

The HTTP status code for each response indicates success or common errors.

| HTTP status code | Description | Possible reason |
|------------------|-------------|-----------------|
| 200 | OK | The request was successful; the response body is an audio file. |
| 400 | Bad Request | A required parameter is missing, empty, or null. Or, the value passed to either a required or optional parameter is invalid. A common issue is a header that is too long. |
| 401 | Unauthorized | The request is not authorized. Check to make sure your subscription key or token is valid and in the correct region. |
| 415 | Unsupported Media Type | It's possible that the wrong `Content-Type` was provided. `Content-Type` should be set to `application/ssml+xml`. |
| 429 | Too Many Requests | You have exceeded the quota or rate of requests allowed for your subscription. |
| 502 | Bad Gateway    | Network or server-side issue. May also indicate invalid headers. |

If the HTTP status is `200 OK`, the body of the response contains an audio file in the requested format. This file can be played as it's transferred, saved to a buffer, or saved to a file.

## Next steps

- [Create a free Azure account](https://azure.microsoft.com/free/cognitive-services/)
- [Asynchronous synthesis for long-form audio](./long-audio-api.md)
- [Get started with custom neural voice](how-to-custom-voice.md)
