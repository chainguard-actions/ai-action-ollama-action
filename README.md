<p align="center">
  <img alt="ollama" height="200px" src="https://raw.githubusercontent.com/ai-action/assets/master/logos/ollama.svg">
</p>

# ollama-action

[![version](https://badgen.net/github/release/ai-action/ollama-action)](https://github.com/ai-action/ollama-action/releases)
[![test](https://github.com/ai-action/ollama-action/actions/workflows/test.yml/badge.svg)](https://github.com/ai-action/ollama-action/actions/workflows/test.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

🦙 Run [Ollama](https://ollama.com/) large language models (LLMs) with GitHub Actions.

## Quick Start

```yaml
# .github/workflows/ollama.yml
on: push
jobs:
  ollama:
    runs-on: ubuntu-latest
    steps:
      - name: Run LLM
        uses: ai-action/ollama-action@v1
        id: llm
        with:
          model: llama3.2
          prompt: Explain the basics of machine learning.

      - name: Get response
        env:
          response: ${{ steps.llm.outputs.response }}
        run: echo "$response"
```

## Usage

Run a prompt against a [model](https://ollama.com/library):

```yaml
- uses: ai-action/ollama-action@v1
  id: explanation
  with:
    model: tinyllama
    prompt: "What's a large language model?"

- name: Get response
  env:
    response: ${{ steps.explanation.outputs.response }}
  run: echo "$response"
```

See [action.yml](action.yml)

## Inputs

### `model`

**Required**: The language [model](https://ollama.com/library) to use.

```yaml
- uses: ai-action/ollama-action@v1
  with:
    model: llama3.2
```

### `prompt`

**Required**: The input prompt to generate the text from.

```yaml
- uses: ai-action/ollama-action@v1
  with:
    prompt: Tell me a joke.
```

To set a multiline prompt:

```yaml
- uses: ai-action/ollama-action@v1
  with:
    prompt: |
      Tell me
      a joke.
```

### `version`

**Optional**: The [Ollama version](https://github.com/ai-action/setup-ollama#version). See the available [versions](https://github.com/ollama/ollama/releases).

```yaml
- uses: ai-action/ollama-action@v1
  with:
    version: 0.5.12
```

## Outputs

### `response`

The generated response message.

```yaml
- uses: ai-action/ollama-action@v1
  id: answer
  with:
    model: llama3.2
    prompt: What's 1+1?

- name: Get response
  env:
    response: ${{ steps.answer.outputs.response }}
  run: echo "$response"
```

> [!NOTE]
> The environment variable is wrapped in double quotes to preserve newlines.

## License

[MIT](LICENSE)

## Privacy

This Action contacts Chainguard's licensing server to verify authorization. Connection metadata (IP address, GitHub repository identifier, timestamp, and any metadata encoded in the auth token) is transmitted to Chainguard, Inc. even if authorization is denied in accordance with our [Privacy Notice](https://www.chainguard.dev/legal/privacy-notice)
