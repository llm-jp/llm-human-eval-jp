# 日本語を対象としたLLMの大規模人手評価


## リポジトリの概要
本リポジトリでは、404件の日本語プロンプトに対する10種類のLLMの大規模人手評価の結果を公開しています。詳細は、文献[1]をご参照ください。


## データの構造

形式は jsonl、各行は下記の構造をしています。

```json
{
	"id": "...###...",
	"turns": [
	  {
	    "prompt": "...",
	    "responses": {
	      "llm_a": {
	        "response": "...",
	        "human_evals": {
	          "detailedness": [1, 1, 1],
	          "factuality": [1, 1, 1],
	          "fluency": [1, 1, 1],
	          "relevance": [1, 1, 1],
	          "overall": [1, 1, 1],
	        }
	      },
	      "llm_b": {
	        ...
	      }
	    }
	  },
	  {
	    "prompt": "...",
	    "responses": {
	      ...
	    }
	  }
	]
}
```

- `"id"`: プロンプトの抽出元データセット名 (後述する「プロンプトのソース」を参照) とそのIDの組み合わせ (`###`により区切られています)。
- `"turns"`:  LLM との対話。各対話は、辞書形式で表現され、対話の系列がリスト形式で格納されています。ほとんどの事例はマルチターンの対話ではないため要素数1ですが、マルチターンの場合には複数の要素が入ります（本データセットでは、最大2です）。
    - `"prompt"` : プロンプト。
    - `"responses"` : LLMの応答。LLMの名称 (省略系) をキーとした (`"llm_a"`, `"llm_b"`, …, `"llm_l"`) 辞書となっています。各LLMの応答は、辞書形式で表現されています。
        - `"response"` : LLMの応答。
        - `"human_evals"` : 各評価軸 (detailedness, factuality, fluency, relevance, overall) における評価結果。辞書形式で格納されています。
            - 各評価軸には、3人の評価者の点数 (1-5) がリスト形式で格納されています。
            - 状況によって、点数が NA であることや、0 であることもありえます (詳しくは文献[1]をご覧ください)。

LLMの名称の省略形 `llm_a` 〜 `llm_l` は、下記のモデルに対応しています。

```python
system_maps = {
    "llm_g": "meta-llama/Meta-Llama-3-8B-Instruct",
    "llm_j": "tokyotech-llm/Llama-3-Swallow-8B-Instruct-v0.1",
    "llm_f": "tokyotech-llm/Llama-3-Swallow-70B-Instruct-v0.1",
    "llm_i": "elyza/Llama-3-ELYZA-JP-8B",
    "llm_b": "llm-jp/llm-jp-13b-instruct-full-ac_001_16x-dolly-ichikara_004_001_single-oasst-oasst2-v2.0",
    "llm_e": "llm-jp/llm-jp-13b-instruct-full-dolly-ichikara_004_001_single-oasst-oasst2-v2.0",
    "llm_c": "cyberagent/calm3-22b-chat",
    "llm_a": "openai/gpt-35-turbo-1106",
    "llm_h": "openai/gpt-4-0613",
    "llm_d": "anthropic/anthropic.claude-3-5-sonnet-20240620-v1:0",
}
```


## サンプルコード

以下に、Python による評価データへのアクセス例を示します。

```python
>>> import json
>>> llm_human_eval_jp = [json.loads(x) for x in open("/path/to/llm-human-eval-jp.jsonl")]
>>> llm_human_eval_jp[0]["id"]
elyza###0

>>> llm_human_eval_jp[0]["turns"][0]["prompt"]
仕事の熱意を取り戻すためのアイデアを5つ挙げてください。

>>> llm_human_eval_jp[0]["turns"][0]["responses"]["llm_a"]["response"]
もちろんです。以下に、仕事の熱意を取り戻すためのいくつかのアイデアをご紹介します。...

>>> llm_human_eval_jp[0]["turns"][0]["responses"]["llm_a"]["human_evals"]
{"relevance": [4, 4, 5], "fluency": [5, 3, 5], "detailedness": [5, 5, 3], "factuality": [5, 5, 4], "overall": [5, 4, 4]}

>>> llm_human_eval_jp[0]["turns"][0]["responses"]["llm_e"]["response"]
- なぜこの仕事を選んだのか、この仕事を始めたきっかけをもう一度考える。...

>>> llm_human_eval_jp[0]["turns"][0]["responses"]["llm_e"]["human_evals"]
{"relevance": [4, 3, 5], "fluency": [4, 2, 2], "detailedness": [4, 3, 5], "factuality": [5, 5, 5], "overall": [4, 3, 3]}

```


## プロンプトのソース

評価に用いたプロンプトは、下記のデータセットの一部を利用しています。

- ichikara-instruction-eval-001
- Elyza: https://huggingface.co/datasets/elyza/ELYZA-tasks-100
- Japanese MT Bench: https://github.com/Stability-AI/FastChat/tree/jp-stable/fastchat/llm_judge/data/japanese_mt_bench
- VicunaQA日本語: https://github.com/ku-nlp/ja-vicuna-qa-benchmark/tree/main/data/jp_bench
- Rakuda: https://huggingface.co/datasets/yuzuai/rakuda-questions


## 論文

本データセットを利用した研究を出版をする場合には、文献[1]の引用をお願いいたします。

[1] 井之上直也, 安藤まや, 後藤美智子, 関根聡, 中山功太, 宮尾祐介. 日本語を対象としたLLMの大規模人手評価. 言語処理学会第31回年次大会論文集, 4 pages, March 2025.
