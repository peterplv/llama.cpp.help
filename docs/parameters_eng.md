
**Disclaimer**:
<br>
This is not a **llama.cpp** manual. It provides a brief overview of the main commands and parameters of **llama.cpp**: how they work, what they are used for, their possible values, and usage examples. It is not a comprehensive guide to all **llama.cpp** features and workflows. This reference is provided "as is", makes no claim to be fully complete or entirely accurate, and is intended only to help both beginners and experienced users. It is recommended for anyone working with local LLMs in **llama.cpp** (including **llama-cli**, **llama-completion**, and **llama-server**).
<br>

Last validation date: **18.11.2026**.

Last used version of **llama.cpp**:

```bash
llama version
version: 0.8.0-dev (build 12014, commit aa588a346)
built with GNU 11.4.0 for Linux x86_64
```
<br>

### ----- common params -----

**-h,    --help, --usage**<br>
print usage and exit

Description:<br>
Shows help for all main parameters: short and long parameter names, brief descriptions, possible values, default values, OS environment variable names for some parameters, and, in some cases, links to the project’s GitHub with a description or discussion of specific parameters.
<br>
<br>

**--version**<br>
show version and build info

Description:<br>
Shows information about the current version and the specific build of llama.cpp.<br>
For example, you can check whether the current build supports CUDA.
<br>
<br>

**--license** (the parameter is absent in newer versions)<br>
show source code license and dependencies

Description:<br>
Shows license information for the main llama.cpp build and for other packages used in llama.cpp.
<br>
<br>

**-cl,   --cache-list**<br>
show list of models in cache

Description:<br>
Shows the list of models in the local cache. Models are downloaded and cached when the "-mu, --model-url MODEL_URL" parameter is used.<br>
If you download models yourself and load them by direct path on local disk, this list will most likely be empty and you do not need this parameter.
<br>
<br>

**--completion-bash**<br>
print source-able bash completion script for llama.cpp

Description:<br>
Prints a bash script to the screen. After you run it, the terminal can suggest llama-cli parameters, complete flags with TAB, and so on.
<br>
<br>

**-t,    --threads N**<br>
number of CPU threads to use during generation<br>
(default: -1)<br>
(env: LLAMA_ARG_THREADS)

Description:<br>
Number of CPU threads used for generation. In theory, the more threads, the higher the speed.

Note:<br>
Generation is limited by memory bandwidth, so beyond a certain number of threads the gain almost disappears, and extra threads can even slow generation down. Test this on your specific configuration and model.<br>
It is also commonly suggested to set the number of physical cores for Ryzen processors, and the number of logical cores for Intel processors.

Default: "--threads -1" (auto).

env: LLAMA_ARG_THREADS - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-tb,   --threads-batch N**<br>
number of threads to use during batch and prompt processing<br>
(default: same as --threads)

Description:<br>
Number of CPU threads used for processing the input prompt and token batches (prefill). If not set, the value of "--threads" is used.<br>
In general, it works like "--threads", but "--threads" applies to token generation and to llama.cpp in general, while this parameter applies only to prefill.

Note:<br>
Generation is limited by memory bandwidth, so beyond a certain number of threads the gain almost disappears. Prompt processing scales well with the number of threads, so in theory "--threads-batch" works best with more threads than the general "--threads", but this should be tested on your specific configuration and model.

Default: "--threads-batch -1" (auto).
<br>
<br>

**-C,    --cpu-mask M**<br>
CPU affinity mask: arbitrarily long hex. Complements cpu-range<br>
(default: "")

Description:<br>
Sets which CPU cores to use via a bitmask in hexadecimal format.<br>
Example: "--cpu-mask 0xF" (0xF = 1111, use cores CPU0, CPU1, CPU2, CPU3).

Default: not used (mask not set).
<br>
<br>

**-Cr,   --cpu-range lo-hi**<br>
range of CPUs for affinity. Complements --cpu-mask

Description:<br>
Sets which CPU cores to use as a range. This is an alternative way to pin threads to specific CPU cores within a given range. Unlike "--cpu-mask", it is specified in regular decimal format. Both parameters can be used together: "--cpu-mask" + "--cpu-range".<br>
Example: "--cpu-range 0-7" (cores 0 to 7, 8 in total).

Default: not used (mask not set).
<br>
<br>

**--cpu-strict <0|1>**<br>
use strict CPU placement<br>
(default: 0)

Description:<br>
Literally "use strict CPU placement". Works together with "--threads/--cpu-mask/--cpu-range". Without strict mode, the threads specified by "--threads" can move freely between CPU cores, as determined by the OS. With strict, each thread is pinned to a separate core.<br>
Possible benefits: lower latency and better cache usage. The main benefit of pinning is that threads are not moved between CPUs and do not lose their cache. However, in practice, the difference in latency is very small in most cases. Strict placement can also sometimes hurt: for example, if another process occupies one of the pinned CPUs, the thread cannot move to a free CPU and has to wait.

Default: "--cpu-strict 0" (disabled).
<br>
<br>

**--prio N**<br>
set process/thread priority: low(-1), normal(0), medium(1), high(2), realtime(3)<br>
(default: 0)

Description:<br>
Priority of the llama.cpp process in the OS process scheduler. Higher priority allows llama.cpp to get more CPU time than other processes in the system. It only matters when multiple processes compete for CPU: on an idle system there is almost no difference.

Possible values:<br>
- (-1) low - background mode, llama.cpp yields to other processes.<br>
- 0 (normal, default) - llama.cpp competes with other processes equally. The best choice in most cases.<br>
- 1 (medium) - llama.cpp gets slightly more CPU time than other processes.<br>
- 2 (high) - llama.cpp gets significantly more CPU time, other processes get less.<br>
- 3 (realtime) - llama.cpp gets the highest priority. System processes may start to lag, so this is not recommended.

The medium(1), high(2), and realtime(3) priorities may require administrator privileges, otherwise llama.cpp will simply ignore this setting.

Default: "--prio 0" (normal).
<br>
<br>

**--poll <0...100>**<br>
use polling level to wait for work
(0 - no polling, default: 50)

Description:<br>
Defines how long a thread actively waits for new work after finishing the current operation (polling).

Examples:<br>
"--poll 0" - the thread sleeps immediately. Minimal CPU load, but waking up takes slightly longer.<br>
"--poll 100" - the longest active waiting. Wake-up is almost instant, but CPU load and power consumption are higher.

The difference is usually very small, milliseconds per token, and may be noticeable only with small, fast models.

Default: "--poll 50".
<br>
<br>

**-Cb,   --cpu-mask-batch M**<br>
CPU affinity mask: arbitrarily long hex. Complements cpu-range-batch<br>
(default: same as --cpu-mask)

Description:<br>
Same as "--cpu-mask", but for batches. Sets which CPU cores to use via a bitmask in hexadecimal format.<br>
Example: "--cpu-mask-batch 0xF" (0xF = 1111, use cores CPU0, CPU1, CPU2, CPU3).

Default: not used (mask not set).
<br>
<br>

**-Crb,  --cpu-range-batch lo-hi**<br>
ranges of CPUs for affinity. Complements --cpu-mask-batch

Description:<br>
Same as "--cpu-range", but for batches. An alternative way to pin threads to specific CPU cores within a given range. Unlike "--cpu-mask-batch", it is specified in regular decimal format. Both parameters can be used together: "--cpu-mask-batch" + "--cpu-range-batch".

Default: not used (not set).
<br>
<br>

**--cpu-strict-batch <0|1>**<br>
use strict CPU placement<br>
(default: same as --cpu-strict)

Description:<br>
Literally "use strict CPU placement". Works together with "--threads-batch/--cpu-mask-batch/--cpu-range-batch". With "--cpu-strict-batch 0", the "--threads-batch" threads can move freely between CPU cores as the OS decides. With "--cpu-strict-batch 1", each thread is pinned to a separate core.<br>
Possible benefits: lower latency and better cache usage. The main benefit of pinning is that threads are not moved between CPUs and do not lose their cache. However, in practice, the difference in latency is very small in most cases. Strict placement can also sometimes hurt: for example, if another process occupies one of the pinned CPUs, the thread cannot move to a free CPU and has to wait.

Default: "--cpu-strict-batch 0" (disabled).
<br>
<br>

**--prio-batch N**<br>
set process/thread priority: 0-normal, 1-medium, 2-high, 3-realtime<br>
(default: 0)

Description:<br>
Same as "--prio", but for prompt processing and batch threads (prefill). The higher the priority, the more CPU time llama.cpp batch threads get when competing with other processes in the system. It only matters when there is competition for CPU: on an idle system there is almost no difference.<br>
In theory, in high(2) mode llama.cpp prefill may run slightly faster, but other processes in the OS will run slower.<br>
In realtime(3) mode, llama.cpp prefill will preempt all other processes in the system, including system processes, potentially making the OS extremely sluggish.

Possible values:<br>
- 0 (normal, default) - llama.cpp prefill competes with other processes equally. The best choice in most cases.<br>
- 1 (medium) - llama.cpp prefill gets slightly more CPU time than other processes.<br>
- 2 (high) - llama.cpp prefill gets significantly more CPU time, other processes get less.<br>
- 3 (realtime) - llama.cpp prefill gets the highest priority. System processes may become unresponsive, so this is not recommended.

The medium(1), high(2), and realtime(3) priorities may require administrator privileges, otherwise llama.cpp will simply ignore this setting.

Default: "--prio-batch 0" (normal).
<br>
<br>

**--poll-batch <0|1>**<br>
use polling to wait for work<br>
(default: same as --poll)

Description:<br>
Same as "--poll", but for batches. Defines how long a batch thread actively waits for new work after finishing the current operation (polling).

Examples:<br>
--poll-batch 0 - the thread sleeps immediately. Minimal CPU load, but waking up takes slightly longer.<br>
--poll-batch 100 - the longest active waiting. Wake-up is almost instant, but CPU load and power consumption are higher.<br>

The difference is usually very small, milliseconds per token, and may be noticeable only with small, fast models.

Default: "--poll-batch 50".
<br>
<br>

**-c,    --ctx-size N**<br>
size of the prompt context<br>
(default: 0 - loaded from model)<br>
(env: LLAMA_ARG_CTX_SIZE)

Description:<br>
Context size. Usually specifies the model's context length (if enough memory is available). The larger the context size, the more memory is required. It often makes sense to use a smaller context size, for example, for short chats or one-off requests such as function/tool calls.

Default: "--ctx-size 0" (the model’s value is used).

env: LLAMA_ARG_CTX_SIZE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-n,    --predict, --n-predict N**<br>
number of tokens to predict<br>
(default: -1, -1 = infinity)<br>
(env: LLAMA_ARG_N_PREDICT)

Description:<br>
Specifies the maximum number of tokens the model will generate. Once this limit is reached, text generation stops abruptly. If this parameter is not set, or "--predict -1" is specified (default), generation will continue indefinitely or until an end-of-generation token is produced (if the model supports one).<br>
Only newly generated tokens are counted; prompt tokens are not included.

Note:<br>
The end token (EOS/EOG) is a special token that ends texts during training, which is how the model learns when to stop generating. End tokens can differ between models. Examples of end tokens: <|endoftext|>, <end_of_turn>, </s>, <|im_end|>, etc.

Default: "--n-predict -1" (indefinitely or until the end token).

Important:<br>
For llama-completion, this parameter has one more possible value:<br>
"-2 = until context filled". Generation stops when the context size is reached (see "--ctx-size").<br>

It is not clear why this value is available only in llama-completion. It may be due to how the tools work: llama-cli and llama-server usually run in conversation mode (their backend is designed for it), while llama-completion works only in single-message generation mode (one-shot call).

env: LLAMA_ARG_N_PREDICT - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-b,    --batch-size N**<br>
logical maximum batch size<br>
(default: 2048)<br>
(env: LLAMA_ARG_BATCH)

Description:<br>
Batch size in tokens. Used for the prefill (prompt, message, or chain of messages). If the prefill is large, it is split into batches that are processed sequentially. Splitting into batches is needed to save memory during computation.

Default: "--batch-size 2048".

env: LLAMA_ARG_BATCH - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-ub,   --ubatch-size N**<br>
physical maximum batch size<br>
(default: 512)<br>
(env: LLAMA_ARG_UBATCH)

Description:<br>
Works like "--batch-size", but sets the "physical" batch size (micro-batch) that is actually sent to the CPU/GPU.<br>
For example, if "--batch-size 2048" and "--ubatch-size 512", the logical batch of 2048 tokens is split into physical batches of 512 tokens, which are sent to the CPU/GPU for computation.

Important: the "--ubatch-size" value must not exceed "--batch-size". Otherwise the parameter has no effect: the original batch size is set by "--batch-size", and "--ubatch-size" only splits the batch before it is sent to the CPU/GPU, so it cannot be larger than the original batch.

Default: "--ubatch-size 512".

env: LLAMA_ARG_UBATCH - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--keep N**<br>
number of tokens to keep from the initial prompt<br>
(default: 0, -1 = all)

Description:<br>
Number of tokens from the initial prompt to keep when the context overflows. From the remaining part, the first half (oldest tokens) is removed.<br>
When the context ("--ctx-size") is full, llama.cpp starts discarding tokens from the beginning of the conversation. This is called context shift. The "--keep" parameter preserves the beginning of the conversation, for example the entire system prompt, so truncation of the context window starts only after the specified number of tokens.

Example:
\[n_past, n_left, n_discard - internal llama.cpp variables, which are computed at runtime\]<br>

--ctx-size 4096, --keep 200, the context is full (n_past = 4096).<br>
n_left = 4096 - 200 = 3896 (the first 200 tokens are kept).<br>
n_discard = 3896 / 2 = 1948 (from the remaining 3896, we remove half - the earlier 1948 tokens).<br>
Each time the context (n_past) fills up again, the context shift is repeated.<br>

If "--keep -1", the entire input prompt is kept.

Default: "--keep 0" (nothing is kept).
<br>
<br>

**--swa-full**<br>
use full-size SWA cache<br>
(default: false)

Description:<br>
Some models, such as Gemma2/3, use Sliding Window Attention (SWA) instead of full attention over the entire context. By default, llama.cpp optimizes the SWA cache and stores only the required (limited) attention window, which significantly saves memory. If "--swa-full" is set, a full-size SWA cache is used without this optimization. A full SWA cache can theoretically give better quality and better compatibility/stability, but requires significantly more memory to store the entire KV cache.

Default: not used.
<br>
<br>

**-fa,   --flash-attn \[on|off|auto\]**<br>
set Flash Attention use ('on', 'off', or 'auto', default: 'auto')<br>
(env: LLAMA_ARG_FLASH_ATTN)

Description:<br>
Enables or disables Flash Attention (FA). FA is a more efficient method of computing attention. It is faster and uses less memory (especially with large contexts), but it can produce slightly different results (better or worse; in most cases the difference is not noticeable).<br>
Parameter "--flash-attn" must also be enabled if V cache quantization ("--cache-type-v") is used during model inference. It is also recommended, but not required, when K cache quantization ("--cache-type-k") is used. For details on KV cache quantization ("--cache-type-k" and "--cache-type-v"), see below.

Default: "--flash-attn auto" (auto).

env: LLAMA_ARG_FLASH_ATTN - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-p,    --prompt PROMPT**<br>
prompt to start generation with; for system message, use -sys

Description:<br>
Prompt text, i.e. the first message from the user. Not to be confused with the System prompt, which is set with the "-sys/--system-prompt" and "-sysf/--system-prompt-file" parameters.
<br>
<br>

**--perf, --no-perf**<br>
whether to enable internal libllama performance timings
(default: false)

Description:<br>
Enables internal performance timings of the libllama library (llama.cpp): load time, prompt eval time, eval time, tokens/s, etc.<br>
Can be used for benchmarking or debugging.

Possible values:<br>
"--perf" - enabled (default).<br>
"--no-perf" - disabled.<br>

Default: "--perf" (enabled).
<br>
<br>

**-f,    --file FNAME**<br>
a file containing the prompt<br>
(default: none)

Description:<br>
Loads the prompt from a file. Used instead of passing the prompt directly in the llama.cpp request with the "--prompt" parameter.

Default: not used.
<br>
<br>

**-bf,   --binary-file FNAME**<br>
binary file containing the prompt<br>
(default: none)

Description:<br>
Loads the prompt from a file in binary mode, byte by byte as is, without any text conversion.<br>
Rarely used. It may be needed to pass raw data, or if the source text has an uncommon encoding or contains special characters, so that the prompt is passed exactly as is.

Default: not used.
<br>
<br>

**-e,    --escape, --no-escape**<br>
whether to process escapes sequences (\\n, \\r, \\t, \\', \\", \\)<br>
(default: true)

Description:<br>
Enables or disables processing of escape sequences. Applies to the prompt text and some other string parameters.<br>
For example, -p "hello\\nworld" becomes:<br>
hello<br>
world<br>

If "--no-escape" is set, escape sequences are passed as plain text.<br>
For example, -p "hello\\nworld" stays as the text: "hello\\nworld".

Default: enabled.
<br>
<br>

#### RoPE parameters
> RoPE (Rotary Positional Embeddings) - a positional encoding mechanism. To understand what these parameters do, you first need to understand how RoPE works. In short, there is a technique for extending a model’s context window. For example, instead of the original 8192 tokens, the model can be made to work with a context of 32768 tokens or more.

**--rope-scaling {none,linear,yarn}**<br>
RoPE frequency scaling method, defaults to linear unless specified by the model<br>
(env: LLAMA_ARG_ROPE_SCALING_TYPE)

Description:<br>
Context scaling method. Options:<br>
- none - no scaling.<br>
- linear - simple linear context stretching.<br>
- yarn - higher-quality scaling for long contexts.<br>

Default: "--rope-scaling linear" (linear, unless the model specifies otherwise).

env: LLAMA_ARG_ROPE_SCALING_TYPE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--rope-scale N**<br>
RoPE context scaling factor, expands context by a factor of N<br>
(env: LLAMA_ARG_ROPE_SCALE)

Description:<br>
Factor by which the context is scaled using RoPE scaling.<br>
For example, "--rope-scale 4". If the model was trained with a context of 8192 tokens, it will be able to work with a context of 32768 tokens (x4).

Important: since the model was not originally trained on a 32768 context, its quality will be lower than that of a model originally trained on a 32768 context.

env: LLAMA_ARG_ROPE_SCALE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.	
<br>
<br>

**--rope-freq-base N**<br>
RoPE base frequency, used by NTK-aware scaling<br>
(default: loaded from model)<br>
(env: LLAMA_ARG_ROPE_FREQ_BASE)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Fine-tuning of the RoPE base frequency. Changes the distribution of positional frequencies and the behavior of Attention on long contexts. Used in NTK scaling and long-context tuning. Makes a long context more stable and preserves Attention quality over long distances. Larger values extend the working context, but slightly reduce the distinction between nearby positions.

For example, "--rope-freq-base 500000". A model with an original context of 8192, extended to a context of 32768 tokens, will work better on long contexts, and positions beyond the original training range (8192) will remain distinguishable. The value "--rope-freq-base 500000" is only an example and must be chosen empirically for each model separately.

Default: loaded from model (the model’s value is used).

env: LLAMA_ARG_ROPE_FREQ_BASE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--rope-freq-scale N**<br>
RoPE frequency scaling factor, expands context by a factor of 1/N<br>
(env: LLAMA_ARG_ROPE_FREQ_SCALE)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Fine-tuning of RoPE for long context. Multiplies position indices by N (linear interpolation), so a long sequence fits into the position range the model was originally trained on. This makes it possible to artificially extend the context during inference.

For example, with "--rope-freq-scale 0.25", a model with an original context of 8192, extended to a context of 32768 tokens, will work on long contexts roughly as if it had been trained on a 32768 context (very roughly). Values that are too small can reduce Attention quality and the understanding of local context.

Important: since the model was not originally trained on a 32768 context, its quality will be lower than that of a model originally trained on a 32768 context.

env: LLAMA_ARG_ROPE_FREQ_SCALE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--yarn-orig-ctx N**<br>
YaRN: original context size of model<br>
(default: 0 - model training context size)<br>
(env: LLAMA_ARG_YARN_ORIG_CTX)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Fine-tuning of RoPE context extension using the YaRN method ("--rope-scaling yarn"). Sets the original context length of the model, for example "--yarn-orig-ctx 8192". From this value, YaRN determines which frequency dimensions to interpolate and which not.

Default: "--yarn-orig-ctx 0" (the model’s value is used).

env: LLAMA_ARG_YARN_ORIG_CTX - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--yarn-ext-factor N**<br>
YaRN: extrapolation mix factor<br>
(default: -1.00, 0.0 = full interpolation)<br>
(env: LLAMA_ARG_YARN_EXT_FACTOR)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Controls how YaRN extends the context.

Possible values:<br>
- 0.0 - interpolation only.<br>
- >0 - more extrapolation.<br>
- 1.0 - maximum extrapolation for high-frequency dimensions.<br>

Intermediate values give a mix (for example, "--yarn-ext-factor 0.5" gives something in between). It is recommended to use automatic mode (-1).

Default: "--yarn-ext-factor -1" (calculated automatically).

env: LLAMA_ARG_YARN_EXT_FACTOR - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--yarn-attn-factor N**<br>
YaRN: scale sqrt(t) or Attention magnitude<br>
(default: -1.00)<br>
(env: LLAMA_ARG_YARN_ATTN_FACTOR)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Adjusts the Attention strength when the context is extended. When the context is extended, Attention becomes more spread out, and YaRN tries to compensate for this. It is recommended to use automatic mode (-1).

Default: "--yarn-attn-factor -1" (calculated automatically).

env: LLAMA_ARG_YARN_ATTN_FACTOR - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--yarn-beta-slow N**<br>
YaRN: high correction dim or alpha<br>
(default: -1.00)<br>
(env: LLAMA_ARG_YARN_BETA_SLOW)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Threshold for low-frequency dimensions of the positional encoding, which are responsible for long-range dependencies in the text. YaRN fully compresses the dimensions below this threshold, as in linear scaling. The larger the value, the more dimensions are compressed.<br>
Approximate value range: 1-32. It is recommended to use automatic mode (-1).

Default: "--yarn-beta-slow -1" (calculated automatically).

env: LLAMA_ARG_YARN_BETA_SLOW - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--yarn-beta-fast N**<br>
YaRN: low correction dim or beta<br>
(default: -1.00)<br>
(env: LLAMA_ARG_YARN_BETA_FAST)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Threshold for high-frequency dimensions of the positional encoding, which are responsible for precise distinction between neighboring tokens. YaRN does not modify the dimensions above this threshold, so local precision is preserved. The smaller the value, the more dimensions remain unmodified.<br>
Approximate value range: 1-32. It is recommended to use automatic mode (-1).

Default: "--yarn-beta-fast -1" (calculated automatically).

env: LLAMA_ARG_YARN_BETA_FAST - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-kvo,  --kv-offload, -nkvo, --no-kv-offload**<br>
whether to enable KV cache offloading<br>
(default: enabled)<br>
(env: LLAMA_ARG_KV_OFFLOAD)

Description:<br>
Place the KV cache in VRAM (GPU) or RAM (CPU).

Options:<br>
"-kvo", "--kv-offload" - place the KV cache in VRAM (GPU).<br>
"-nkvo", "--no-kv-offload" - place the KV cache in RAM (CPU).<br>
Keeping the KV cache in VRAM usually increases generation speed. Recommended if a GPU is available.

Default: enabled.

env: LLAMA_ARG_KV_OFFLOAD - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--repack, -nr, --no-repack**<br>
whether to enable weight repacking<br>
(default: enabled)<br>
(env: LLAMA_ARG_REPACK)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Repack the model weights into a more efficient format optimized for CPU SIMD instructions (AVX2, AVX512, NEON, etc.). Affects only CPU mode.

Options:<br>
"--repack" - faster inference, but slower loading of model weights into RAM.<br>
"-nr", "--no-repack" - slower inference, but faster loading of model weights into RAM (weights are read directly from the file without repacking).<br>
Recommended to keep enabled.

Default: enabled.

env: LLAMA_ARG_REPACK - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--no-host**<br>
bypass host buffer allowing extra buffers to be used<br>
(env: LLAMA_ARG_NO_HOST)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Disables the default host buffer (a buffer in RAM used for faster data transfer between RAM and VRAM) and allows the backend to use other buffer types to store model weights. Buffer types may differ in compatibility, performance, or availability.<br>
Parameter "--no-host" is rarely used in most cases.

env: LLAMA_ARG_NO_HOST - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-ctk,  --cache-type-k TYPE**<br>
KV cache data type for K<br>
allowed values: f32, f16, bf16, q8_0, q4_0, q4_1, iq4_nl, q5_0, q5_1<br>
(default: f16)<br>
(env: LLAMA_ARG_CACHE_TYPE_K)

Description:<br>
Data type for caching K (Attention Key) values. The default is "f16" (float16). You can set a quantized type, "bf16" (bfloat16), or the more precise "f32" (float32). "f32" requires twice as much memory for the cache and most likely will not noticeably improve text generation quality. Smaller (quantized) types use less memory, but reduce generation quality and may run slower because of the extra dequantization of values.<br>
For some tasks, quantized K cache types such as "q8_0" perform about as well as the original "f16", but in other tasks the quality loss may be more noticeable. "q4_*" quantization types reduce generation quality more.

According to some reports, quantizing the K cache reduces generation quality less noticeably than quantizing the V cache.

Important! If you compile llama.cpp with CUDA support, you need to specify the following option to enable all quantization types:<br>
-DGGML_CUDA_FA_ALL_QUANTS=ON.

Default: "--cache-type-k f16" (float16).

env: LLAMA_ARG_CACHE_TYPE_K - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-ctv,  --cache-type-v TYPE**<br>
KV cache data type for V<br>
allowed values: f32, f16, bf16, q8_0, q4_0, q4_1, iq4_nl, q5_0, q5_1<br>
(default: f16)<br>
(env: LLAMA_ARG_CACHE_TYPE_V)

Description:<br>
Data type for caching V (Attention Value) values. The default is "f16" (float16). You can set a quantized type, "bf16" (bfloat16), or the more precise "f32" (float32). "f32" requires twice as much memory for the cache and most likely will not noticeably improve text generation quality. Smaller (quantized) types use less memory, but reduce generation quality and may run slower because of the extra dequantization of values.<br>
For some tasks, quantized V cache types such as "q8_0" perform about as well as the original "f16", but in other tasks the quality loss may be more noticeable. "q4_*" quantization types reduce generation quality more.

According to some reports, quantizing the V cache reduces generation quality more than quantizing the K cache. Flash Attention ("-fa/--flash-attn") must be enabled to quantize the V cache.

Important! If you compile llama.cpp with CUDA support, you need to specify the following option to enable all quantization types:<br>
-DGGML_CUDA_FA_ALL_QUANTS=ON.

Default: "--cache-type-v f16" (float16).

env: LLAMA_ARG_CACHE_TYPE_V - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-dt,   --defrag-thold N**<br>
KV cache defragmentation threshold (DEPRECATED)<br>
(env: LLAMA_ARG_DEFRAG_THOLD)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

KV cache fragmentation threshold above which memory cell defragmentation is triggered. The parameter is deprecated and no longer used in newer versions of llama.cpp.

How it worked: the KV cache consists of memory cells, one cell per token. When tokens are removed or shifted (context shift, sequence removal, a request finishing in one of the slots, etc.), gaps remain in the cache. Previously, a new batch required a contiguous block of free cells, and defragmentation moved the occupied cells to the beginning and compacted the cache. The "-dt" threshold set the fraction of gaps in memory above which defragmentation was triggered. In newer versions of llama.cpp, the cache can place tokens in non-contiguous cells, so defragmentation is no longer needed. The parameter is kept for compatibility.

env: LLAMA_ARG_DEFRAG_THOLD - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-np,   --parallel N**<br>
number of parallel sequences to decode (default: 1)<br>
(env: LLAMA_ARG_N_PARALLEL)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Number of parallel slots for processing requests simultaneously. Required for handling multiple requests in parallel. Not used in normal single mode.

Default: "--parallel 1".

env: LLAMA_ARG_N_PARALLEL - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--rpc SERVERS**<br>
comma-separated list of RPC servers (host:port)<br>
(env: LLAMA_ARG_RPC)

Description:<br>
Connects RPC servers for distributed (remote) inference. For each server, specify its IP address and port; multiple servers can be specified separated by commas. Example:<br>
192.168.1.10:50051,192.168.1.11:50052.

By default, model weights and the KV cache are distributed across all devices (local and remote) in proportion to their available memory. The proportions can be set manually with the "--tensor-split" parameter.

If both the server and the client support RDMA (RoCEv2 on Linux or RDMA over Thunderbolt 5 on macOS), the transport is negotiated automatically. To use it, specify the address of the RDMA-capable interface in "--rpc". A connection through any other interface will stay on TCP.

All llama.cpp instances (main and RPC) must be built with the "-DGGML_RPC=ON" option.

> Important! The RPC backend is at the proof-of-concept stage, and its functionality is neither reliable nor secure. Do not run RPC servers on open networks or in environments containing sensitive data.

For more details about the RPC implementation in llama.cpp, see:<br>
[https://github.com/ggml-org/llama.cpp/blob/master/tools/rpc/README.md](https://github.com/ggml-org/llama.cpp/blob/master/tools/rpc/README.md)

env: LLAMA_ARG_RPC - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-lm,   --load-mode MODE**<br>
model loading mode.<br>
- auto: mmap, unless a device does not support it.<br>
- none: no special loading mode.<br>
- mmap: memory-map model (if mmap disabled, slower load but may reduce pageouts if not using mlock).<br>
- mlock: force system to keep model in RAM rather than swapping or compressing.<br>
- mmap+mlock: mmap + force system to keep model in RAM rather than swapping or compressing.<br>
- dio: use DirectIO if available.<br>
(default: auto)<br>
(env: LLAMA_ARG_LOAD_MODE)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Selects how the model file is loaded into memory. Replaces the old parameters "--mmap", "--no-mmap", "--mlock" and "--direct-io".

Available options:<br>
- auto (default) - uses mmap if the device supports it.<br>
- none - no special mode is used: llama.cpp reads the entire model file from disk into allocated RAM. Loading takes longer, but the model resides entirely in memory and there are no repeated disk accesses. Equivalent to the old "--no-mmap".<br>
- mmap - the model file is mapped into the address space of the llama.cpp process, and the OS loads the required pages into RAM (page cache) as they are accessed. Gives a faster start and near-instant subsequent launches, but if RAM is insufficient, the OS may evict some pages and have to reread them from disk later. Equivalent to the old "--mmap".<br>
- mlock - prevents the OS from swapping the model out of RAM to SWAP (page file) and from compressing it. Fewer possible freezes and more stable speed, but requires enough memory. Equivalent to the old "--no-mmap --mlock" (the model is read into RAM and locked there).<br>
- mmap+mlock - the model is mapped via mmap, and all its pages are locked in RAM. The most predictable option in terms of speed when there is enough memory. Equivalent to the old "--mmap --mlock".<br>
- dio - DirectIO: reads the model file from disk directly, bypassing the OS page cache. Works if DirectIO is supported by the OS and file system. Equivalent to the old "--direct-io".<br>

Default: "--load-mode auto" (auto).

env: LLAMA_ARG_LOAD_MODE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-lzm,  --lazy-mode MODE**<br>
on-demand reading of certain tensors, for example per-layer embeddings<br>
- on: read the rows of such tensors from disk on demand instead of keeping them resident (requires mmap).<br>
- auto: on, but only for tensors larger than 4 GiB.<br>
- off: always keep them resident.<br>
(default: auto)<br>
(env: LLAMA_ARG_LAZY_MODE)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Literally: reading specific tensors on demand, for example, per-layer embeddings.<br>
The required tensor row is read from disk only when it is accessed, so the whole tensor is not permanently held in RAM. This saves memory, but each access waits for the data to load from disk.

Available options:<br>
- on - read such tensors on demand instead of keeping them in memory permanently (requires mmap, "--load-mode mmap").<br>
- auto (default) - same as "on", but only for tensors larger than 4 GiB. Smaller tensors are always kept in memory.<br>
- off - always keep such tensors fully in memory.<br>

env: LLAMA_ARG_LAZY_MODE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--mlock**<br>
In newer versions of llama.cpp, this parameter was moved to "-lm, --load-mode" as one of its modes.
<br>
<br>

**--mmap, --no-mmap**<br>
In newer versions of llama.cpp, this parameter was moved to "-lm, --load-mode" as one of its modes.
<br>
<br>

**-dio,  --direct-io, -ndio, --no-direct-io**<br>
In newer versions of llama.cpp, this parameter was moved to "-lm, --load-mode" as one of its modes.
<br>
<br>

**--numa TYPE**<br>
attempt optimizations that help on some NUMA systems<br>
- distribute: spread execution evenly over all nodes<br>
- isolate: only spawn threads on CPUs on the node that execution started on<br>
- numactl: use the CPU map provided by numactl<br>
if run without this previously, it is recommended to drop the system page cache before using this<br>
see [https://github.com/ggml-org/llama.cpp/issues/1437](https://github.com/ggml-org/llama.cpp/issues/1437)<br>
(env: LLAMA_ARG_NUMA)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Optimization for NUMA systems (NUMA - Non-Uniform Memory Access, a memory architecture in multiprocessor systems).

Available options:<br>
- distribute - distributes threads evenly across NUMA nodes.<br>
- isolate - threads are created only on the CPUs of the node where the process started.<br>
- numactl - uses the CPU affinity mask set via the numactl utility.<br>

If llama.cpp was previously run without this parameter, it is recommended to drop the system page cache before using it. For more details, see:<br>
[https://github.com/ggml-org/llama.cpp/issues/1437](https://github.com/ggml-org/llama.cpp/issues/1437).

Parameter "--numa" used on multi-socket servers; not needed for normal use.

env: LLAMA_ARG_NUMA - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-dev,  --device <dev1,dev2,..>**<br>
comma-separated list of devices to use for offloading (none = don't offload)<br>
use --list-devices to see a list of available devices<br>
(env: LLAMA_ARG_DEVICE)

Description:<br>
Devices to use for model offloading, separated by commas. The list of available devices can be viewed with the "--list-devices" parameter. If set to "none", offloading is not used at all and the model stays on CPU/RAM. If no values are specified, the default available devices are used.

env: LLAMA_ARG_DEVICE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--list-devices**<br>
print list of available devices and exit

Description:<br>
Shows the list of devices available in the system for model offloading.<br>
Running llama.cpp with this parameter prints the device list and exits.
<br>
<br>

**-ot,   --override-tensor \<tensor name pattern>=\<buffer type>,...**<br>
override tensor buffer type<br>
(env: LLAMA_ARG_OVERRIDE_TENSOR)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Literally: override the tensor buffer type.<br>
The parameter allows you to manually specify which memory (which device) to store specific model tensors in. Rules are set by tensor name patterns; if there are several rules, they are separated by commas. The pattern is a regular expression matched against tensor names, and the buffer type is specified by a device name: CPU, CUDA0, Vulkan0, and so on. The list of available devices can be obtained with "--list-devices".

Why this is needed. For example, with the "--n-gpu-layers" parameter, whole layers are offloaded to the GPU. Offloading individual tensors with "--override-tensor" gives finer control: different parts of a layer can be placed on different devices. This can be useful, for example, for MoE models. In them, most of the weights are expert tensors, but only a small part of them is active at any given moment during token generation. Therefore, in some cases it makes sense to:<br>
- keep on faster devices what is used constantly and has a large effect on speed: attention, normalization, shared layers, etc.<br>
- move expert tensors to slower devices (for example, CPU), since they take up a lot of memory, but only some of them are active at a time.<br>
Note: this is just an example. In practice, various devices can be used, for example different GPUs installed in the system, different backends (Cuda, Vulkan, Metal, etc.), NPU, RPC, etc. Tensors can be distributed in completely different proportions and across different devices, depending on the specific configuration and the models used.

env: LLAMA_ARG_OVERRIDE_TENSOR - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-cmoe, --cpu-moe**<br>
keep all Mixture of Experts (MoE) weights in the CPU<br>
(env: LLAMA_ARG_CPU_MOE)

Description:<br>
For MoE models (Mistral, DeepSeek, etc). Keeps all expert weights in RAM (CPU) instead of VRAM (GPU). This frees up GPU memory, but processing on the CPU is significantly slower.

env: LLAMA_ARG_CPU_MOE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-ncmoe, --n-cpu-moe N**<br>
keep the Mixture of Experts (MoE) weights of the first N layers in the CPU<br>
(env: LLAMA_ARG_N_CPU_MOE)

Description:<br>
For MoE models (Mistral, DeepSeek, etc.). Unlike "-cmoe, --cpu-moe", which places all expert layers in RAM (CPU), this parameter sets how many of the first N expert layers are placed in RAM (CPU), which reduces VRAM (GPU) usage. A trade-off between VRAM usage and speed.

env: LLAMA_ARG_N_CPU_MOE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--moe-cache-mib N**<br>
GPU cache size in MiB for the MoE experts kept in the CPU.<br>
with multiple GPUs, it is split among them like the layers (--tensor-split)<br>
(default: 0 - disabled)<br>
(env: LLAMA_ARG_MOE_CACHE_MIB)

Description:<br>
This option is only used with MoE models. For experts which loaded into RAM (via "-ncmoe" or "-cmoe"), it allocates a cache of the specified size in VRAM. The experts themselves stay in slower RAM (CPU), but the cache runs in faster VRAM (GPU). The larger the cache, the higher the speed, but the less VRAM remains for the context (KV cache). The size is set in MiB (mebibytes).

If the system has multiple GPUs, the cache is distributed between them in the same way as model layers, according to "--tensor-split".

Default: "--moe-cache-mib 0" (not used).

env: LLAMA_ARG_MOE_CACHE_MIB - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-ncffn, --n-cpu-ffn N**<br>
keep the dense FFN weights of the first N layers in the CPU<br>
(dense models; for MoE expert weights use --n-cpu-moe)<br>
(env: LLAMA_ARG_N_CPU_FFN)

Description:<br>
Similar to "-ncmoe, --n-cpu-moe", but for dense models (classic non-MoE models, such as GPT3, Llama3, Gemma2, and others).<br>
For FFN (Feed Forward Network) weights, sets how many of the first N model layers have their FFN weights placed in RAM (CPU). The remaining layers stay in VRAM (GPU).

env: LLAMA_ARG_N_CPU_FFN - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-ngl,  --gpu-layers, --n-gpu-layers N**<br>
max. number of layers to store in VRAM, either an exact number, 'auto', or 'all'<br>
(default: auto)<br>
(env: LLAMA_ARG_N_GPU_LAYERS)

Description:<br>
The number of model layers to place on the GPU (VRAM). The value is either a number, ‘all’ (all layers), or ‘auto’ (determined automatically).<br>
Placing model layers on the GPU significantly improves performance. If the entire model is placed on the GPU, the speedup compared to the CPU is the largest (often several times faster).

Default: "--gpu-layers auto" (auto).

env: LLAMA_ARG_N_GPU_LAYERS - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-sm,   --split-mode {none,layer,row}**<br>
how to split the model across multiple GPUs, one of:<br>
- none: use one GPU only<br>
- layer (default): split layers and KV across GPUs<br>
- row: split rows across GPUs<br>
(env: LLAMA_ARG_SPLIT_MODE)

Description:<br>
How to split the model across multiple GPUs. Possible values:<br>
- none - use a single GPU.<br>
- layer - model layers and the KV cache are distributed across GPUs.<br>
- row - weights are split by rows across GPUs, and each GPU computes its part of every layer. Intermediate results and the KV cache are kept on the main GPU.<br>

If the system has multiple identical GPUs with a fast interconnect (NVLink), "row" mode can theoretically increase generation speed.
On PCIe without NVLink, "row" mode is likely to be slower than "layer" because of constant data exchange between the GPUs.

Default: "--split-mode layer".

env: LLAMA_ARG_SPLIT_MODE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-ts,   --tensor-split N0,N1,N2,...**<br>
fraction of the model to offload to each GPU, comma-separated list of proportions, e.g. 3,1<br>
(env: LLAMA_ARG_TENSOR_SPLIT)

Description:<br>
How to distribute the model across GPUs, using specified proportions.

Examples:<br>
"--tensor-split 2,1,1" - sum is 4, so: GPU0 = 2/4 = 50%, GPU1 = 1/4 ~ 25%, GPU2 = 1/4 ~ 25%.<br>
"--tensor-split 4,2,1" - sum is 7, so: GPU0 = 4/7 ~ 57%, GPU1 = 2/7 ~ 29%, GPU2 = 1/7 ~ 14%.<br>

env: LLAMA_ARG_TENSOR_SPLIT - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-mg,   --main-gpu INDEX**<br>
the GPU to use for the model (with split-mode = none), or for intermediate results and KV (with split-mode = row)<br>
(default: 0)<br>
(env: LLAMA_ARG_MAIN_GPU)

Description:<br>
Related to the "-sm, --split-mode" parameter. Sets the index of the main GPU, which is used to hold the model (with "--split-mode none") or for intermediate results and the KV cache (with "--split-mode row"). For example: "--main-gpu 1" - if the system has multiple GPUs, llama.cpp treats the GPU with index 1 as the main one (indexing starts at 0). This GPU is used for the most important part of the workload.

This can be useful, for example, if the GPUs in the system have different performance or different amounts of VRAM. In that case, it makes sense to choose the faster GPU or the one with more VRAM as the main GPU.

Default: "--main-gpu 0" (the first GPU is used).

env: LLAMA_ARG_MAIN_GPU - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-fit,  --fit \[on|off\]**<br>
whether to adjust unset arguments to fit in device memory ('on' or 'off')<br>
(default: 'on')<br>
(env: LLAMA_ARG_FIT)

Description:<br>
Helps when there is not enough memory and the model cannot be loaded. Automatically adjusts some arguments so that the model fits in memory (usually VRAM). Works together with the "--fit-target" and "--fit-ctx" parameters.

Possible values:<br>
- on - automatically adjust parameters to fit.<br>
- off - do not adjust parameters automatically; if there is not enough memory, model loading fails with an error.<br>

Default: "--fit on" (enabled).

env: LLAMA_ARG_FIT - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-fitt, --fit-target MiB0,MiB1,MiB2,...**<br>
target margin per device for --fit, comma-separated list of values, single value is broadcast across all devices<br>
(default: 1024)<br>
(env: LLAMA_ARG_FIT_TARGET)

Description:<br>
Sets the amount of memory (in MiB) to leave free. Works together with the "--fit" parameter.<br>
If the system has multiple devices, specify the values separated by commas. If a single value is given, it is applied to all devices.<br>
Used as a safety margin, to keep some free memory on each device after the model and context are placed.<br>
For example: "--fit-target 2048" - leave 2048 MiB (2 GiB) free.

Default: "--fit-target 1024".

env: LLAMA_ARG_FIT_TARGET - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-fitc, --fit-ctx N**<br>
minimum ctx size that can be set by --fit option<br>
(default: 4096)<br>
(env: LLAMA_ARG_FIT_CTX)

Description:<br>
Sets the minimum context size (in tokens) that "--fit" cannot reduce the context below when saving memory. Works together with the "--fit" parameter.<br>
For example: "--fit-ctx 4096". With "--ctx-size 32768", if there is not enough memory and "--fit on" is enabled, llama.cpp can automatically reduce the context size, but not below 4096 tokens.

Default: "--fit-ctx 4096".

env: LLAMA_ARG_FIT_CTX - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--check-tensors**<br>
check model tensor data for invalid values<br>
(default: false)

Description:<br>
Checks tensors for invalid values (NaN, Inf, etc.) when loading the model. Helps detect a corrupted model, but may slow down loading.

Default: not used (disabled).
<br>
<br>

**--override-kv KEY=TYPE:VALUE,...**<br>
advanced option to override model metadata by key. to specify multiple overrides.<br>
either use comma-separated values. types: int, float, bool, str.<br>
example: --override-kvtokenizer.ggml.add_bos_token=bool:false,tokenizer.ggml.add_eos_token=bool:false<br>

Description:<br>
Overrides model metadata without modifying the original GGUF file.<br>
GGUF files store all of the model’s associated data: configuration, tokenizer, special token information, and so on. Everything is in a single file together with the weights, unlike, for example, Hugging Face transformers models format. This parameter allows you to override the model’s metadata (settings) at launch, without editing the GGUF file.

Examples:<br>
"--override-kv tokenizer.ggml.add_bos_token=bool:false" - the model used to add the BOS token from the GGUF metadata, now it will not (may be useful for experiments).<br>
"--override-kv llama.context_length=int:32768" - if the "context_length" value was set incorrectly, you can fix it when loading the model, without touching the GGUF file itself.
<br>
<br>

**--op-offload, --no-op-offload**<br>
whether to offload host tensor operations to device<br>
(default: true)

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Whether to offload tensor operations to the GPU when the weights are stored in RAM.<br>
Details: when the weights are stored in RAM (CPU) but the computations are heavy, such as matrix multiplications when processing large batches (prompt processing), they are still executed on the GPU: the weights are temporarily transferred there for the duration of the operation. This is enabled by default. The "--no-op-offload" parameter disables this, and all operations stay on the CPU. This is needed, for example, if transferring weights over PCIe cancels out the speed gain (slow bus), or in some other cases.

Default: "--op-offload" (enabled).
<br>
<br>

**--lora FNAME**<br>
path to LoRA adapter (use comma-separated values to load multiple adapters)

Description:<br>
Applies a LoRA adapter to the model. Specify the path to the LoRA file. Multiple adapters can be specified, separated by commas.

> Note: LoRA is a lightweight fine-tuning method that changes the model’s behavior without full fine-tuning.
<br>
<br>

**--lora-scaled FNAME:SCALE,...**<br>
path to LoRA adapter with user defined scaling<br>
(format: FNAME:SCALE,...) note: use comma-separated values

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Applies a LoRA adapter to the model with a specified scale. Works like "--lora", but with an additional scale value.<br>
Specify the adapter path followed by its scale, separated by a colon. Multiple LoRAs with their scales can be specified, separated by commas. Their effects are combined.

SCALE: the working range is approximately "-2.0" to "2.0", but is not limited to these values; 0 means no effect. In practice, values from "0.5" to "1.0" are typically used. Values above 1 may cause distortion. For example: "0.5" - weaker LoRA effect; "1.0" - medium (standard) effect; "1.5" - noticeably stronger effect, may cause distortion. With a negative value, the LoRA effect may be unpredictable; not recommended.
<br>
<br>

**--control-vector FNAME**<br>
add a control vector<br>
note: use comma-separated values to add multiple control vectors

Description:<br>
Applies a control vector to change the model’s behavior. Specify the path to the file. Multiple vectors can be specified, separated by commas.

> Note: control vectors change the model’s hidden states during inference, unlike LoRA adapters, which change the model’s weights. Both parameters serve a similar purpose, adjusting the model’s behavior, but use different methods.
<br>
<br>

**--control-vector-scaled FNAME:SCALE,...**<br>
add a control vector with user defined scaling SCALE<br>
(format: FNAME:SCALE,...) note: use comma-separated values

Description:<br>
\[Note: the description of this parameter may not be accurate\]

Applies a control vector to the model with a specified scale. Works like "--control-vector", but with an additional scale value.<br>
Specify the path to the control vector followed by its scale, separated by a colon. Multiple control vectors with their scales can be specified, separated by commas. Their effects are combined.

SCALE: the working range is approximately "-2.0" to "2.0", but is not limited to these values; 0 means no effect. In practice, values from "0.3" to "1.0" are typically used. Values above 1 may cause distortion. For example: "0.3" - weaker effect; "1.0" - medium (standard) effect; "1.5" - noticeably stronger effect, may cause distortion. With a negative value, the effect may be unpredictable; not recommended.
<br>
<br>

**--control-vector-layer-range START END**<br>
layer range to apply the control vector(s) to, start and end inclusive

Description:<br>
Limits the model layers to which control vectors are applied (the "--control-vector" or "--control-vector-scaled" parameters).<br>
START is the first layer, END is the last layer. Layer numbering starts at 0.<br>
Example: "--control-vector-layer-range 10 20" - control vectors are applied only to layers 10 through 20, inclusive.<br>
This parameter applies globally to all loaded control vectors.
<br>
<br>

**-m,    --model FNAME**<br>
model path to load<br>
(env: LLAMA_ARG_MODEL)

Description:<br>
Loads a model from a local file. Specify the relative or absolute path to the model file.

env: LLAMA_ARG_MODEL - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-mu,   --model-url MODEL_URL**<br>
model download url<br>
(default: unused)<br>
(env: LLAMA_ARG_MODEL_URL)

Description:<br>
Downloads a model from the specified URL and loads it. The model is stored in the cache, which can be viewed with the "--cache-list" command.

Default: not used.

env: LLAMA_ARG_MODEL_URL - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-dr,   --docker-repo \[\<repo\>\/\]\<model\>\[:quant\]**<br>
Docker Hub model repository. repo is optional, default to ai\/.<br>
quant is optional, default to :latest.<br>
example: gemma3<br>
(default: unused)<br>
(env: LLAMA_ARG_DOCKER_REPO)

Description:<br>
Loads a model from Docker Hub. Optionally, a specific "repo" can be specified (default is "ai/"). The quantization type can also be specified. If no quantization is specified, ":latest" (the default model variant) is used.<br>
Example: "--docker-repo ai/gemma3:Q4_K_M".

Default: not used.

env: LLAMA_ARG_DOCKER_REPO - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-hf,   -hfr, --hf-repo \<user\>\/\<model\>\[:quant\]**<br>
Hugging Face model repository; quant is optional, case-insensitive,<br>
default to Q4_K_M, or falls back to the first file in the repo if Q4_K_M doesn't exist.<br>
mmproj is also downloaded automatically if available. to disable, add --no-mmproj<br>
example: unsloth/phi-4-GGUF:q4_k_m<br>
(default: unused)<br>
(env: LLAMA_ARG_HF_REPO)

Description:<br>
Loads a model from Hugging Face. Specify the repository owner (user), the model name (model), and the required quantization (quant, case-insensitive). If no quantization type is specified, "Q4_K_M" is used; if this quantization is not available in the repository, the first available variant is used.<br>
The "mmproj" file is also downloaded automatically (if present). To skip downloading it, use the "--no-mmproj" parameter.

> Note: "mmproj" is a projector for multimodal models (models that understand images/video/speech in addition to text). It converts the encoder output (for example, from a vision encoder) into the LLM embedding space. As a result, an image is converted into a sequence of vectors that the LLM processes the same way as embeddings of regular text tokens.

Example: "--hf-repo unsloth/phi-4-GGUF:Q6_K".

Default: not used.

env: LLAMA_ARG_HF_REPO - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-hff,  --hf-file FILE**<br>
Hugging Face model file. If specified, it will override the quant in --hf-repo<br>
(default: unused)<br>
(env: LLAMA_ARG_HF_FILE)

Description:<br>
Loads a model from Hugging Face by file name. Used together with "--hf-repo".<br>
If this parameter is set, it overrides the quantization type specified in "--hf-repo". That is, this parameter has higher priority.<br>
Example: "--hf-repo unsloth/phi-4-GGUF --hf-file phi-4-Q6_K.gguf".

Default: not used.

env: LLAMA_ARG_HF_FILE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-hfv,  -hfrv, --hf-repo-v \<user\>\/\<model\>\[:quant\]** (the parameter is absent in newer versions)<br>
Hugging Face model repository for the vocoder model<br>
(default: unused)<br>
(env: LLAMA_ARG_HF_REPO_V)

Description:<br>
Loads a vocoder model from Hugging Face. Specify the repository owner (user), the model name (model), and the required quantization (quant, case-insensitive). If no quantization type is specified, "Q4_K_M" is used; if this quantization is not available in the repository, the first available variant is used.

> Note: Vocoder model converts the audio tokens produced by the main TTS model into sound. It works only together with the main TTS model, which is loaded separately.

Default: not used.

env: LLAMA_ARG_HF_REPO_V - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-hffv, --hf-file-v FILE** (the parameter is absent in newer versions)<br>
Hugging Face model file for the vocoder model<br>
(default: unused)<br>
(env: LLAMA_ARG_HF_FILE_V)

Description:<br>
Loads a vocoder model from Hugging Face by file name. Used together with "--hf-repo-v".

Default: not used.

env: LLAMA_ARG_HF_FILE_V - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-hft,  --hf-token TOKEN**<br>
Hugging Face access token<br>
(default: value from HF_TOKEN environment variable)<br>
(env: HF_TOKEN)

Description:<br>
Hugging Face access token. Required for private repositories and for gated models (where you must accept a license agreement).<br>
Example: "--hf-repo user/private-model --hf-token hf_xxxxxxxxx".

Default: the value is taken from the HF_TOKEN environment variable.

env: HF_TOKEN - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--log-disable**<br>
Log disable

Description:<br>
Disables logging. Model responses and basic service information are not affected.
<br>
<br>

**--log-file FNAME**<br>
Log to file<br>
(env: LLAMA_LOG_FILE)

Description:<br>
Writes logs to a file. Specify the file path and file name, or just the file name.<br>
For example, "--log-file gemma3_q8_test.log" writes the log to the current directory, where the llama.cpp instance was launched.

env: LLAMA_LOG_FILE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--log-jsonl, --no-log-jsonl**<br>
Log as JSONL (one JSON object per line) to stdout, this also disables colored logging<br>
(default: disabled)

Description:<br>
Enables log output in the structured JSONL format (JSON Lines). Logs are written to stdout, and each entry is a separate JSON object on its own line.

JSONL is convenient for log post-processing: each entry is structured (it has a "key" and a "value"), so entries can be filtered, sorted, and searched by field names, and used for other post-processing operations. In a regular text log, each line is plain text with no explicit structure, so detailed analysis requires parsing it with regular expressions. With JSONL this is usually not needed, since the entries are already structured.<br>
When this log format is enabled, colored log output ("--log-colors") is disabled automatically.

Available options:<br>
"--log-jsonl" - JSONL logging is enabled.<br>
"--no-log-jsonl" - JSONL logging is disabled (regular text log format).<br>

Default: "--no-log-jsonl" (disabled).
<br>
<br>

**--log-colors \[on|off|auto\]**<br>
Set colored logging ('on', 'off', or 'auto')<br>
(default: 'auto' - enables colors when output is to a terminal)<br>
(env: LLAMA_LOG_COLORS)

Description:<br>
Colored logs. Available options:<br>
- on - colors are enabled.<br>
- off - colors are disabled.<br>
- auto (default) - colors are enabled only when the output is to a terminal; when logging to a file or a pipe (passing a program’s output as input to another program), ANSI color codes are not added.<br>

Default: "--log-colors auto".

env: LLAMA_LOG_COLORS - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-v,    --verbose, --log-verbose**<br>
Set verbosity level to infinity (i.e. log all messages, useful for debugging)

Description:<br>
Enables verbose logging mode: the log includes the maximum amount of information. Often used for debugging and may be excessive for normal use.
<br>
<br>

**--offline**<br>
Offline mode: forces use of cache, prevents network access<br>
(env: LLAMA_OFFLINE)

Description:<br>
Enables standalone offline mode. The local cache ("--cache-list") is used by force and network access is blocked (for example, for downloading models). That is, models from the cache are used unless specified otherwise.

env: LLAMA_OFFLINE - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**-lv,   --verbosity, --log-verbosity N**<br>
Set the verbosity threshold. Messages with a higher verbosity will be ignored.<br>
Values:<br>
- 0: generic output<br>
- 1: error<br>
- 2: warning<br>
- 3: info<br>
- 4: debug<br>
(default: 1)<br>
(env: LLAMA_LOG_VERBOSITY)

Description:<br>
Log verbosity level. Messages with a higher verbosity level are ignored (not written to the log).

Available options:<br>
- 0 - general information.<br>
- 1 - error messages.<br>
- 2 - warnings.<br>
- 3 - informational messages.<br>
- 4 - full operation details (debug).<br>

Default: "--log-verbosity 1" (general information and error messages).

env: LLAMA_LOG_VERBOSITY - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--log-prefix**<br>
Enable prefix in log messages<br>
(env: LLAMA_LOG_PREFIX)

Description:<br>
Adds a prefix to log lines.<br>
It may look like this:<br>
[INFO] loading model.<br>
[WARNING] low memory.<br>
[ERROR] failed to load model.<br>

env: LLAMA_LOG_PREFIX - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--log-timestamps**<br>
Enable timestamps in log messages<br>
(env: LLAMA_LOG_TIMESTAMPS)

Description:<br>
Adds a timestamp to each log message.

env: LLAMA_LOG_TIMESTAMPS - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--spec-draft-type-k, -ctkd, --cache-type-k-draft TYPE**<br>
KV cache data type for K for the draft model<br>
allowed values: f32, f16, bf16, q8_0, q4_0, q4_1, iq4_nl, q5_0, q5_1<br>
(default: f16)<br>
(env: LLAMA_ARG_SPEC_DRAFT_CACHE_TYPE_K)

Description:<br>
Works like "--cache-type-k", but for the loaded draft model.<br>
Data type for caching K (Attention Key) values. The default is "f16" (float16). You can set a quantized type, "bf16" (bfloat16), or the more precise "f32" (float32). "f32" requires twice as much memory for the cache and most likely will not noticeably improve text generation quality. Smaller (quantized) types use less memory, but reduce generation quality and may run slower because of the extra dequantization of values.<br>
For some tasks, quantized K cache types such as "q8_0" perform about as well as the original "f16", but in other tasks the quality loss may be more noticeable. "q4_*" quantization types reduce generation quality more.

According to some reports, quantizing the K cache reduces generation quality less noticeably than quantizing the V cache.

> Note: Draft model is a small, fast model that works together with the main (large) model. It quickly predicts several next tokens and passes them to the main model for verification. If the prediction is confirmed, the main model accepts these tokens without generating them step by step on its own, which speeds up overall text generation.

Default: "--cache-type-k-draft f16" (float16).

env: LLAMA_ARG_CACHE_TYPE_K_DRAFT - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
<br>
<br>

**--spec-draft-type-v, -ctvd, --cache-type-v-draft TYPE**<br>
KV cache data type for V for the draft model<br>
allowed values: f32, f16, bf16, q8_0, q4_0, q4_1, iq4_nl, q5_0, q5_1<br>
(default: f16)<br>
(env: LLAMA_ARG_SPEC_DRAFT_CACHE_TYPE_V)

Description:<br>
Works like "--cache-type-v", but for the loaded draft model.<br>
Data type for caching V (Attention Value) values. The default is "f16" (float16). You can set a quantized type, "bf16" (bfloat16), or the more precise "f32" (float32). "f32" requires twice as much memory for the cache and most likely will not noticeably improve text generation quality. Smaller (quantized) types use less memory, but reduce generation quality and may run slower because of the extra dequantization of values.<br>
For some tasks, quantized V cache types such as "q8_0" perform about as well as the original "f16", but in other tasks the quality loss may be more noticeable. "q4_*" quantization types reduce generation quality more.

According to some reports, quantizing the V cache reduces generation quality more than quantizing the K cache. Flash Attention ("-fa/--flash-attn") must be enabled to quantize the V cache.

> Note: Draft model is a small, fast model that works together with the main (large) model. It quickly predicts several next tokens and passes them to the main model for verification. If the prediction is confirmed, the main model accepts these tokens without generating them step by step on its own, which speeds up overall text generation.

Default: "--cache-type-v-draft f16" (float16).

env: LLAMA_ARG_CACHE_TYPE_V_DRAFT - an environment variable that can be set in the OS to specify the value of this parameter for the current session.
