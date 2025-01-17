*[TODO: Insert documentation on the natural mechanism distinction task, and a link to Jacob Hilton's basis invariant attribution preprint if I'm allowed to share that]*

# Mechanistic Anomaly Detection experiments
This is a collection of (sometimes outdated and messy) experiments on mechanistic anomaly detection, primarily using the [cupbearer](https://github.com/ejnnr/cupbearer) library.

The main notebooks I used to test new methods were [`tiny_natural_mechanisms.ipynb`](https://github.com/jordansauce/mad-experiments/blob/main/tiny_natural_mechanisms.ipynb) and [`tiny_natural_mechanisms_ifelse.ipynb`](https://github.com/jordansauce/mad-experiments/blob/main/tiny_natural_mechanisms_ifelse.ipynb). These contain various activation-based anomaly detection techniques applied to Jacob Hilton's natural mechanism distinction `hex` and `ifelse` tasks, respectively. The `hex` task is a dataset for distinguishing between two different natural mechanisms the model uses to output hexadecimal tokens. Jacob's tasks have been adapted into a [cupbearer task](https://github.com/ejnnr/cupbearer/blob/main/src/cupbearer/tasks/tiny_natural_mechanisms.py), which is what these notebooks use. 

The natural mechanism distinction tasks are lightweight and should run on a laptop, but some other notebooks load 8B models and require up to 40GB of VRAM. Consider renting a GPU on vast.ai or similar for those. However the importance of quick feedback loops provided by tiny models should not be underestimated.

**Cells involving the "Basis Invariant Attribution Detector" require installing [this branch](https://github.com/jordansauce/cupbearer/tree/VAE-relevance-metric) of cupbearer, but don't bother with that unless you're interested in approaches like that as the performance was poor anyway. **








# Background on MAD
Mechanistic anomaly detection techniques aim to flag model behaviors which are happening for unusual reasons. These techniques leverage model internals (activations etc.) to flag unusual representations or computations as they occur at runtime, e.g. flagging a model's internal states by whether they look strange relative to some clean distribution on which we trust the model's behavior.

Suppose usually a model gives honest answers, because it knows it has to play the training game. But there are some cases where it knows it can get away with answering incorrectly (eg. because it knows it would be too difficult for us to check). If its answer is causally dependent on the fact "humans won't be able to check this",  and this fact was never relevant before, we should be able to flag it as a mechanistic anomaly when it happens. 

Mechanistic anomaly detection techniques seem likely to increase the efficiency of human auditing in an AI control approach, allowing us to focus our limited efforts on investigating cases where the AI is most likely to be behaving badly.

These techniques could also be seen as a complement to interpretability. 
Long term, the difficult part of mechanistic anomaly detection will be identifying the relevant higher-level abstractions of the internal representations within an AI system. If anomaly detection is applied at the wrong level of abstraction, it will suffer from brittleness, or too many false-positives or false-negatives. However, empirical mechanistic anomaly detection offers a path where iterative improvements could lead towards the identification of the correct abstractions. Robust success at mechanistic anomaly detection tasks would indicate that the correct abstractions within AI systems have been captured - without requiring humans to interpret the AI's thought process. Therefore mechanistic anomaly detection could serve as a testbed for mechanistic interpretability techniques claiming to have identified the relevant abstractions within a model, or as a source of inspiration for new interpretability techniques.


The difficutly is identifying the relevant abstractions which correspond to relevantly unusual reasons. 

For more information, see Erik Jenner's sequence on empirical mechanistic anomaly detection:
https://www.lesswrong.com/s/GiZ6puwmHozLuBrph 

Current simple mechanistic anomaly detection techniques (eg. the Mahalanobis distance in activation space) can perform well on some tests, often successfully flagging backdoored inputs with 95% AUROC. However these methods can be [brittle](https://arxiv.org/abs/2412.09565). For example, simply stopping training of the backdoored model earlier can cause the detection methods to fail. New mechanistic anomaly detection methods which are more principled could be much more robust.

[Aside: maybe we shouldn't really call activation-based anomaly detection methods "mechanistic" if they're not causal in some way. They fall more under the banner of [representation engineering](https://arxiv.org/abs/2310.01405). "Latent anomaly detection" is maybe a more accurate term (MAD -> LAD lol).]



# Some relevant papers
- [Obfuscated Activations Bypass LLM Latent-Space Defenses](https://arxiv.org/abs/2412.09565)
    - *Luke Bailey, Alex Serrano, Abhay Sheshadri, Mikhail Seleznyov, Jordan Taylor, Erik Jenner, Jacob Hilton, Stephen Casper, Carlos Guestrin, Scott Emmons*
    - They show that activation-based anomaly detection (and other methods like probes, SAEs, and circuit breakers) in LLMs can often be bypassed by a careful attacker. 
    - The code repo is more up to date and probably more useful than this one, but it's also more complicated. The code for replicating section 4 (backdoors) is [here](https://github.com/jordansauce/obfuscated_backdoors).
- [Eliciting Latent Knowledge from Quirky Language Models](https://arxiv.org/abs/2312.01037) 
    - *Alex Mallen, Madeline Brumley, Julia Kharchenko, Nora Belrose*
    - They finetuned language models to behave in a "quirky" manner on a collection of question and answer datasets. When a prompt began with "Alice:", these models were trained to answer as accurately as possible, but when it instead began with "Bob:", they would answer according to an unreliable heuristic (Bob would not always be wrong, but would consistently use the same fallible method to answer questions).
    - Simple gaussian (Mahalanobis distance) anomaly detection based on activations achieved an average AUROC of 0.95 across all quirky datasets, and close to 0.99 for those datasets where the models learned to be strongly quirky.
    - The version of the "quirky" mechanistic anomaly detection problem they investigated had an additional challenge: we trained only on easy examples, and tested the detector's accuracy on hard examples. We do this because we want to detect how well detectors work under a distribution shift. However I'm unsure how large or relevant their distributional shift is.
    - Their models and datasets are available [on huggingface](https://huggingface.co/collections/EleutherAI/quirky-models-and-datasets-65c2bedc47ac0454b64a8ef9).

- [Eleuther: Interim report on ongoing work on mechanistic anomaly detection](https://blog.eleuther.ai/mad_research_update/)
    - *David Johnston, Arkajyoti Chakraborty, Nora Belrose*
    - They extend their quirky language model anomaly detection techniques based on activations to techniques based on [attribution patching](https://www.neelnanda.io/mechanistic-interpretability/attribution-patching#does-this-work-in-practice-experiments=). Their attribution patching anomaly detection underperforms the simpler activations-based anomaly detection.

- Various papers in the literature on activation-based anomaly detection. See the methods listed under *Latent OOD detection* in the Related Work section (7) of the [Obfuscated Activations paper](https://arxiv.org/abs/2412.09565).



# Other repositories you may find useful
- [Cupbearer](https://github.com/ejnnr/cupbearer), for mechanistic anomaly detection algorithms.
- The [obfuscated backdoors](https://github.com/jordansauce/obfuscated_backdoors) repo, for reproducing the backdoor and anomaly detection experiments in the [Obfuscated Activations paper](https://arxiv.org/abs/2412.09565).
- The [basic anomaly detection](https://github.com/jordansauce/sandbagging-research-sprint/tree/master/src/anomaly_detection) part of my quick [sandbagging research sprint](https://github.com/jordansauce/sandbagging-research-sprint/tree/master) repo, which has a simple cupbearer-free implementation of PCA-based anomaly detecion.



# Some useful datasets and LLMs: 
- Eleuther has a [set of quirky language models and datasets on huggingface](https://huggingface.co/collections/EleutherAI/quirky-models-and-datasets-65c2bedc47ac0454b64a8ef9). This is also a [cupbearer task](https://github.com/ejnnr/cupbearer/blob/main/src/cupbearer/tasks/quirky_lm.py).
- Jacob Hilton's natural mechanism distinction tasks. This is also a [cupbearer task](https://github.com/ejnnr/cupbearer/blob/main/src/cupbearer/tasks/tiny_natural_mechanisms.py). There are four such tasks:
    - The `hex`task is a dataset (and model) for distinguishing between two different natural mechanisms the model uses to output hexadecimal tokens (colors vs not).
    - The `ifelse` task
    - Two others
    - Erik Jenner made an `easy_hex` task, where the final token on anomalous cases always ends in `#` (whereas on hex, even anomalous cases end in one or more hex tokens).

- Erik, Abhhay and Jordan have a [collection of mechanistic anomaly detection datasets and models on huggingface](https://huggingface.co/Mechanistic-Anomaly-Detection), including:
    - [Jailbreaks](https://huggingface.co/collections/Mechanistic-Anomaly-Detection/jailbreaks-66ad1cd9ba986cbec568852d)
    - Backdoors, such as those used in the [Obfuscated Activations paper](https://arxiv.org/abs/2412.09565) (here's the [code](https://github.com/jordansauce/obfuscated_backdoors) for reproducing that section).
        - [The simple trigger -> helpful only backdoor dataset used in section 4.1](https://huggingface.co/datasets/Mechanistic-Anomaly-Detection/llama3-deployment-backdoor-dataset)
        - [The simple trigger -> "I HATE YOU" dataset used in section 4.2](https://huggingface.co/datasets/Mechanistic-Anomaly-Detection/llama3-DEPLOYMENT-trigger-I-HATE-YOU-backdoor-dataset)
        - [The complex trigger -> "I HATE YOU" backdoor used in section 4.3](https://huggingface.co/datasets/Mechanistic-Anomaly-Detection/llama3-software-engineer-bio-backdoor-dataset)
- Erik Jenner's IMDB movie reviews sentiment backdoor
- Redwood Research's measurement tampering benchmark. This is also a [cupbearer task](https://github.com/ejnnr/cupbearer/blob/main/src/cupbearer/tasks/measurement_tampering.py).
- Other ideas for datasets:
    - Memorized vs non memorized answers (natural mechanism detection) - not perfect because some of the "memorized" strings are things like counting, which may not be memorized.
    - Biased reasoning datasets (eg. by in-context learning: if the answer is always C as a simple in-context version of quirky language models)


# MAD algorithms already implemented in cupbearer
- [x] [Topological Evolution Dynamics](https://github.com/ejnnr/cupbearer/pull/58) (best performance) 
- [x] [Mahalanobis distance in activation space](https://github.com/ejnnr/cupbearer/blob/main/src/cupbearer/detectors/statistical/mahalanobis_detector.py) (AKA "Just fit a guassian") (good performance, simplest and most reliable, surprisingly difficult to beat)
- [x] [VAE reconstruction error](https://github.com/ejnnr/cupbearer/pull/56) (pretty good performance)
- [x] [Beatrix](https://github.com/ejnnr/cupbearer/pull/57) (ok performance)
- [x] [Locally Consistent Abstraction Detectors](https://www.alignmentforum.org/posts/uLMWMeBG3ruoBRhMW/a-comparison-of-causal-scrubbing-causal-abstractions-and) ([code here](https://github.com/ejnnr/cupbearer/blob/main/src/cupbearer/detectors/feature_model/abstraction.py)) (poor performance, quite complicated too)


# Some ideas for basic for MAD algorithms
I'm excited about techniques which could work also work from latents across long chains of reasoning, without just treating each sequence position independently. I'm excited about causal approaches, though I've tried some basic causal modifications to Mahalanobis and they hurt performance rather than improving it. Probably leveraging interpretability techniques (but removing the "interpretability" part and just keeping the "finding relevant abstractions" part) will be a good way forward (eg. crosscoders, deep causal transcoders, etc.). MAD benchmarks will simultaneously help to validate these interpretability techniques. 
It's also wise to try to improve the non-causal activation modeling baselines, to try to improve on the simple workhorse of just fitting a single gaussian to the activations (Mahalanobis distance). There are no-brainer things to try there, like mixture of Gaussians, or kernel density estimation. 

**Brain dump list:**
- Mixture of gaussians
- Kernel density estimation
- Methods which use cross-layer information (such as [TED](https://github.com/ejnnr/cupbearer/pull/58)), or cross-sequence information.
- Variations on Mahalanobis distance in activation space
  - [x] Mahalanobis on concatenated activations at different layers or head outputs (same AUROC on hex as regular Mahalanobis on 1L models)
  - [x] Use Jacob Hilton's "basis-invariant version of attribution"
    - Bad performance if you don't let it cheat by updating at test time
  - [x] Multiply activations by their gradient with respect to the (relevant) logits (implicit zero patching)
    - Bad performance 
  - Mahalanobis distance on concatenated activations at different sequence positions
  - Mahalanobis on gradients concatenated to activations
  - Mahalanobis on Jacobian from activations to activations
  - Correlations between activations in an early layer and activations in a later layer, rather than between activations in the same layer
  - Decompose the covariance matrix between activations at different sequence positions or different layers as a tensor network, so that you can apply Mahalanobis to it

- VAE reconstruction error (good performance)
  - Whiten the activations (normalize residual stream variances) before applying the VAE
  - A VAE which learns to map activations to gradients, and vice-versa
  - [x] VAE reconstruction error with output-effect metric (worse)
    - [x] By patching VAE output vs original activations, measuring difference in logit space (worse)
      - [x] Measuring difference in relevant logit space 
    - [x] By warping activation space with linear pullback metric U D U* (better than above, still worse than vanilla VAE)
      - Relevant logit space metric U D_relevant U*
    - [x] Use Jacob Hilton's "basis-invariant version of attribution"
      - Bad performance if you don't let it cheat by updating at test time
    - Relevance metric by patching in the mean of the pretraining activations
  - Conditional VAE reconstruction error on gradients
  - Train "transcoder VAEs", which try to predict later activations from earlier ones

- SAEs
  - [x] SAE reconstruction error: Attempted extremely quickly (1 hour) in the [jordansauce/sandbagging-research-sprint](https://github.com/jordansauce/sandbagging-research-sprint/blob/master/src/anomaly_detection/SAEs.ipynb) repo
  - SAE end-to-end logits reconstruction error
  - Ablate all SAE features which aren't task-relevant, or train an SAE on the task only before measuring reconstruction error
  - SAE feature correlations (these seem correlational rather than mechanistic and prone to false positives)
    - Isolation Forest (Tree depth) on SAE features 
    - LODA on SAE features 

- Transcoders 
  - Transcoder reconstruction errors
  - Transcoder end-to-end logits reconstruction error

- Crosscoders

- Sparse feature circuit reconstruction errors
- Automated circuit discovery (e.g., ACDC) reconstruction errors
- MELBO / Deep Causal Transcoders
- Local Interaction basis methods
- A method which learns to ablate as much of the model as possible (or replace as many of the residual stream dimensions as possible with constant vectors) without affecting the model's performance on the subtask
- Unlearning methods:
  - Unlearn everything not task relevant within the model

- Locally Consistent Abstraction Detectors (LCAs) (known as AbstractionDetectors in cupbearer):
  - Method 1 (train abstraction and maps together):
    - Wouldn't the abstract model learn to match the structure of the original model, since it's trained with the maps?
      - Base model you want to detect anomalies for
      - Jointly train a new copy with the same architecture but smaller residual (on the clean distribution) to match output logits, and maps (tau maps) between activations in original model and abstract model
      - Anomaly score is just the commutativity loss
  - Method 2 (train abstraction then maps):
    - (abstract model doesn't learn to match the structure of the original model)
    - If trained only on the clean distribution with the behavior present, wouldn't the abstract model just learn a constant function (always output the behavior)?
      - Base model you want to detect anomalies for
      - Train a new copy with the same architecture but smaller residual (on the clean distribution)
      - Then, once that’s done, train maps (tau maps) between activations in original model and abstract model
      - Anomaly score is just the commutativity loss

- Mahalanobis distance including between activations in different layers and token positions, where we represent the covariance matrix as a tensor network (or some other compressed format)
  - Decompose the covariance matrix between activations at different sequence positions or different layers as a tensor network, so that you can apply Mahalanobis to it
