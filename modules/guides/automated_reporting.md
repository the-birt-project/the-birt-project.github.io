# <span class="center-text">Automated Reporting with LLM's</span>
<br><br>
[Back](../guide_menu.md)

## Introduction
BIRT can automate report generation for an investigation. It accomplishes this by leveraging LLM's to translate the MITRE ATT&CK labelled evidence and events into a structured narrative. Each endpoint's evidence, events and analyst comments are processed into individual report summaries and then combined to form one overarching report. Generating these reports requires many API calls and if those API endpoints aren't fully in your control, can expose sensitive data to outside organizations. BIRT can query any OpenAI compatible API endpoint configured with any model and context length. Obviously, the specific model and context length available are **extremely** important to the quality of the output.

Finding a model and configuration that works for you and your team is a bit of an art, now. When determining which model you should acquire and test, good results in Needle-in-the-Haystack tests and coherency at large context windows are the best for this type of detailed work. For example, the LLAMA 3.1 8B model's and Cohere's Command R are not included in this test because they fall apart at high context. LLAMA 3.1 8B will go into a repeat loop (at any temperature) and Command R will go on tangents that are only cut off by the over-context-length protections in BIRT.

This research marks a certain point in time as LLM technology is advancing at a quick pace. When this engine was first written, ChatGPT 3.5 was the SOTA model available and local LLM's were LLAMA 2 derivatives. Every update to BIRT should bring more compatibility and better output alongside new features as the team explores how to integrate this technology into the investigative process.

## Reporting Engine Internals
As of [version 0.9.6.3](https://www.thebirtproject.com "Download the latest version!"), BIRT extracts the global timeline from the investigation and separates it into individual endpoint timelines. Next, each timeline is subjected to analysis and summarization by several task-specific agents. These agents query for contextual data on each MITRE technique and sub-technique relevant to their specific lens on the data.

The query engine attempts to fill the context budget with as much contextual, event and evidence data without going over the limit. In many cases, open-weight models will fail to respond to a prompt, so the engine adjusts temperature, scales back the contextual data and reduces the resolution on the evidence and event data until a response is generated. The individual endpoint timeline summaries are then fused together into task-specific summaries which are used to build an overall investigation summary.

This pipelined approach takes raw events, artifact data and analyst notes from a forensic investigation and iteratively mutates it into a more understandable narrative format. A model's architecture, parameter count, context-length and ability to follow instructions are all important considerations when choosing a model to assist your investigations.

Let's get started.

## Test Setup: The Reasonable Researcher Build
A 5-year-old gaming rig + an extra GPU and +64GB of memory:
- 9900K 96GB RAM
- 2 x Nvidia 3090 48GB VRAM
- [LM Studio 0.2.29](https://lmstudio.ai)

Each model's layers are fully offloaded into GPU memory and the maximum context (with flash attention enabled) is configured.

If you'd like to follow along, [download the latest version](https://www.thebirtproject.com "Download the latest version!") and generate reports for the included APT29 demo investigation with your own API server.
## Results

The PDF's are the output of the reporting engine. The gpt-4o report contains the full evidence timeline with notes, as an example. The remaining PDF's have the timeline truncated to save some bandwidth abd storage, but they're the exact same data. The only difference is the generated summaries for each endpoint and the overall investigation.

Model | Context Length | Output Tokens | Query Time | Notes | PDF
-- | -- | -- | -- | -- | --
OpenAI gpt-4o | 128,000 | 8,888 | ~ 6 mins | - | [PDF](../../assets/report_pdf/GPT_4o_128k.pdf)
OpenAI gpt-4o-mini | 128,000 | 8,374 | ~5 mins | - | [PDF](../../assets/report_pdf/GPT_4o_mini_128k.pdf)
OpenAI gpt-3.5-turbo | 16,384 | 6,933 | ~4 mins | - | [PDF](../../assets/report_pdf/gpt_3.5_turbo_16k.pdf)
Google Gemma 2 IT 27b Q8_0 | 32,000 | 21,546 | ~12 mins | rope_freq_base 160000 to extend context| [PDF](../../assets/report_pdf/gemma_2_it_27b_Q8_0.pdf)
Mistral Large Instruct 2407 IQ2_XXS | 32,768 | 23,081 | ~29 mins | - | [PDF](../../assets/report_pdf/Mistral_Large_Instruct_2407_IQ2_XXS.pdf)
Mistral Large Instruct 2407 IQ2_XS | 32,768 | 25,644 | ~37 mins | - | [PDF](../../assets/report_pdf/Mistral_Large_Instruct_2407_IQ2_XS.pdf)
Mistral Large Instruct 2407 IQ2_M | 17,000 | 27,477 | ~38 mins | - | [PDF](../../assets/report_pdf/Mistral_Large_Instruct_2407_IQ_2M.pdf)
Mixtral 8x7b Instruct Q6_K | 32,768 | 60,877 | ~18 mins | - | [PDF](../../assets/report_pdf/Mixtral_8x7b_Instruct_Q6_K.pdf)
Mixtral 8x7b Instruct Q5_K_M | 32,768 | 33,090 | ~12 mins | - | [PDF](../../assets/report_pdf/Mixtral_8x7b_Instruct_Q5_K_M.pdf)
LLAMA 3.1 70B Q3_K_M | 32,000 | 15,494 | ~19 mins | starting temperature raised to 0.6/0.7 | [PDF](../../assets/report_pdf/llama_3.1_70B_Q3_K_M.pdf)
LLAMA 3.1 70B Q3_K_L | 32,000 | 21,009 | ~22 mins | starting temperature raised to 0.6/0.7 | [PDF](../../assets/report_pdf/llama_3.1_70B_Q3_K_L.pdf)
LLAMA 3.1 70B IQ4_XS | 28,000 | 18,076 | ~17 mins | starting temperature raised to 0.6/0.7 | [PDF](../../assets/report_pdf/llama_3.1_70B_IQ4_XS.pdf)

## Result Observations
LLAMA 3.1 8B, Cohere Command R and several others were tested, but produced repeating text or other undesired output and were omitted from the results.

Overall, they're mostly useable, as-is. Some will need a bit of trimming for awkward formatting or the [common GPT slop](https://github.com/AlpinDale/gptslop). They represent a good start to crafting a narrative from an organized collection of events, artifacts, context and analyst notes.

### OpenAI ChatGPT
The current gold standard. It follows instructions and produces the smoothest and most accurate output. 

### Meta LLAMA 3.1
The output for the various quantization of this model is promising, I believe with more VRAM and a Q4 or Q6 version it'll become more useful. Apart from hallucinations, these models "mumble", they respond with what look like internal instructions or chain-of-thought. I'm not sure if this is a setting/configuration issue, but it leaks into the response and doesn't get filtered out in further grooming passes. This class of models also requires a higher starting temperature for the analysis and report personas. Even with these higher temperatures, failure to respond was the highest of any of the models. I look forward to revisiting this with more VRAM, a higher bpw quantization and longer context.

### Mistral AI
The Mixtral 8x7b models are starting to show their age, hallucinations and failure to follow instructions were not super common, but still prevalent. The Mistral Large models were fantastic, I'll be acquiring some more GPU's and the first model I want to test is the Q4 or better quantization of this model.

### Google Gemma 2
A solid model whose context can be extended to 32k without artifacts. No real frills or deep insights, does what it's told and produces good summarizations.

## Advanced LLM Options
Additional configuration options are available in the file *%APPDATA%\BIRT\data\connections\llm_options.json*.

```json
{
  "inv_summary_analyst_min_response_length": 100,
  "inv_summary_analyst_temperature": 0.1,
  "inv_summary_analyst_timeout": 300,
  "inv_summary_report_min_response_length": 100,
  "inv_summary_report_temperature": 0.2,
  "inv_summary_report_timeout": 300,
  "stream_query_temperature": 0.2,
  "stream_query_timeout": 60,
  "analyst_ctx_budget_divisor": null,
  "report_ctx_budget_divisor": null,
  "debug_llm_output": false
}
```

Currently, the values above represent the defaults that are installed with the application.
- **inv_summary_analyst_min_response_length** - Minimum analyst response tokens allowed.
- **inv_summary_analyst_temperature** - Start analyst temperature, value range 0.0 - 1.0.
- **inv_summary_analyst_timeout** - API query timeout in seconds.
- **inv_summary_report_min_response_length** - Minimum report response tokens allowed.
- **inv_summary_report_temperature** - Start report temperature, value range 0.0 - 1.0.
- **inv_summary_report_timeout** - API query timeout in seconds.
- **stream_query_temperature** - Start temperature for evidence and event summary queries, value range 0.0 - 1.0
- **stream_query_timeout** - API query timeout in seconds.
- **analyst_ctx_budget_divisor** - GPT default 1.5, others 3. Divide context by this number to arrive at budget for queries. Lower numbers give more space for event data but can cause issues with some models.
- **report_ctx_budget_divisor** - GPT default 1.5, others 2. Divide context by this number to arrive at budget for queries. Lower numbers give more space for event data but can cause issues with some models.
- **debug_llm_output** - true/false. Setting to true will print API queries and response to console for debugging purposes.

<br><br>
[Back](../guide_menu.md)


