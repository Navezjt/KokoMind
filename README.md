# KokoMind 

[![License](https://img.shields.io/badge/Code%20License-Apache_2.0-green.svg)](https://github.com/CHATS-lab/KokoMind/blob/main/LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/release/python-390/)

This is the repo for **KokoMind**, a dataset with multi-party social interactions to evaluate LLMs' social understanding abilities. The repo contains:

- The evaluation [data](https://github.com/CHATS-lab/KokoMind/tree/main/data) of social interactions.
- The [code](https://github.com/CHATS-lab/KokoMind/tree/main/eval) for model evaluation.
- Check out the [blog post of KokoMind](https://chats-lab.github.io/KokoMind) to see some demos.

<!-- [[Project Page](https://chats-lab.github.io/KokoMind/)] [Paper] -->

<p align="center">
    <img src="./website/img/gorilla.png" width="15%"> <br>
  Logo of <b>KokoMind</b>.
</p>

## News

- **[2023.07.05]** **KokoMind** is released at https://chats-lab.github.io/KokoMind/.

## Demo
https://github.com/CHATS-lab/KokoMind/assets/13882237/731427bf-0d3c-4870-b36e-e146f954309b

## Dataset

**KokoMind** contains 150 complex multi-party social interactions (50 per source) with free-text questions and answers. To ensure diversity and scalability and avoid data contamination, all the social interactions, questions, and answers are generated by GPT-4 and verified by human experts later. These generations are based on three different sources:

- 🤖 GPT-4-only: This subset is created solely by GPT-4 through prompting, without grounding on existing sources.
- 🎦 Movie-based: To avoid data contamination, this portion of the data is grounded on diverse scenarios pulled from movies released after 2022. GPT-4 shapes these situations, maintaining the core essence while adding its own elements.
- 🧠 ToMi-based: This segment contains data backboned by a simulated dataset, ToMi, which involves moving physical objects to different places, a classic test for theory of mind. These social interactions are again embellished and expanded by GPT-4.

For each social interaction, we ask various questions designed to probe the following aspects of social understanding.

- 🧠 Theory of Mind: Questions evaluating understanding of others' mental states and perspectives.
- 👍 Social Norm: Questions aiming to discern societal values and norms within the situations.
- 😃 Emotion Recognition: Questions targeted at identifying and understanding emotional elements within the context.
- 👨‍👩‍👧 Social Relation: Queries focusing on interpersonal dynamics and relationships.
- 🤔 Counterfactual Questions: Hypothetical queries designed to explore alternative outcomes or possibilities.
- 📝 Social Advice: Questions eliciting advice or action recommendations relevant to the given situation.

`question_nonverbal_yes_v0.1.json` contains 770 samples in total. This [JSON Lines](https://jsonlines.org/) file is a list of dictionaries, with each dictionary contains the following fields:

- `question_id`: int, the unique ID of the question.
- `text`: str, social interaction context and question.
- `answer`: str, GPT-4 answer that has been further verified by human.
- `source`: str, one of the three data sources: `gpt-4`, `movie`, `tomi`.
- `category`: str, one of six question categories: `ToM`, `Social Norm`, `Emotion Recognition`, `Social Relation`, `Counterfactual`, `Social Advice`.

`question_nonverbal_no_v0.1.json` contains the same social interactions and questions but but with the non-verbal cues in the parenthesis (e.g., nervously sipping coffee, etc) removed from the context.

## Evaluation

### Pre-requisite

```bash
pip install -r requirements.txt
export OPENAI_API_KEY=<your_api_key>
export ANTHROPIC_API_KEY=<your_api_key>
```

### Generate model answers

``` bash
# Generate local model anwers
# Use vicuna-7b as an example
python eval/get_model_answer.py --model-path ${PATH_TO_LOCAL_HF_MODEL} --model-id vicuna-7b --question-file data/question_nonverbal_yes_v0.1.jsonl --answer-file data/answer/answer_vicuna-7b.jsonl --num-gpus 8

# GPT-3 answer (reference model by alpaca-eval)
python eval/qa_baseline_gpt3.py -q data/question_nonverbal_yes_v0.1.jsonl -o data/answer/answer_gpt3.jsonl

# GPT-3.5 answer
python eval/qa_baseline_gpt35.py -q data/question_nonverbal_yes_v0.1.jsonl -o data/answer/answer_gpt35.jsonl

# GPT-4.0 answer
python eval/qa_baseline_gpt4.py -q data/question_nonverbal_yes_v0.1.jsonl -o data/answer/answer_gpt4.jsonl

# Claude answer
python eval/qa_baseline_claude.py -q data/question_nonverbal_yes_v0.1.jsonl -o data/answer/answer_claude.jsonl
```

### Run evaluation

Our evaluation is based on [Alpaca-Eval](https://github.com/tatsu-lab/alpaca_eval).

```bash
# Convert to alpaca_eval input format
python eval/generate_alpaca_eval.py -q data/question_nonverbal_yes_v0.1.jsonl -a data/answer/answer_gpt3.jsonl -o data/alpaca_eval/answer_gpt3.json

alpaca_eval make_leaderboard --leaderboard_path data/alpaca_results/leaderboard.csv --all_model_outputs "./data/alpaca_eval/answer_*" --reference_outputs data/alpaca_eval/answer_gpt3.json --is_overwrite_leaderboard True
```

## License

This project is an early-stage research showcase, designed solely for non-commercial purposes. It adheres to [OpenAI's data usage terms](https://openai.com/policies/terms-of-use), and [ShareGPT's privacy practices](https://chrome.google.com/webstore/detail/sharegpt-share-your-chatg/daiacboceoaocpibfodeljbdfacokfjb). Let us know if you spot any potential violations. The software's code is available under the Apache License 2.0.

## Acknowledgement

We would like to thank [Yejin Choi](https://homes.cs.washington.edu/~yejin/) from UW, [Louis-Philippe Morency](https://www.cs.cmu.edu/~morency/) from CMU, [Jason Weston](https://scholar.google.com/citations?user=lMkTx0EAAAAJ&hl=en) from Meta, and [Diyi Yang](https://cs.stanford.edu/~diyiy/) from Stanford for their enlightening dialogues and constructive inputs.

## Citation

Please cite our work if you find it useful.

``` bib
@misc{Shi_KokoMind_Can_Large_2023,
  author = {Shi, Weiyan and Qiu, Liang and Xu, Dehong and Sui, Pengwei and Lu, Pan and Yu, Zhou},
  title = {{KokoMind: Can Large Language Models Understand Social Interactions?}},
  month = jul,
  year = {2023},
  url = {https://chats-lab.github.io/KokoMind/}
}
```
