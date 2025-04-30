# pg_porting_assistant
Oracle to PostgreSQL Query Porting Assistant Utility  
* This utility operates based on [fast-agent](https://github.com/evalstate/fast-agent).  
* It inherits and applies the [Apache-2.0 license](LICENSE) of fast-agent.

## Prerequisites
#### Folder Structure
```bash
- pg_porting_assistant
- fastagent.config.yaml
- fastagent.secrets.yaml # Optional
- ASIS
  └── team
      └── user1.sql
  └── team2
      └── user2.sql

## API / Ollama Configuration
#### Using Commercial API
```bash
# fastagent.secrets.yaml
Register API key
openai:
  api_key: sk-abcdefg ## Enter OpenAI API Key
```

#### Using Ollama
Ollama requires installation and downloading of an LLM model.  
Recommended models for local (general PC) environments are llama3.2:3b and gemma3:4b.  
[Ollama Official Site](https://ollama.com/)  
```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull [model_name]
```

## Usage
```bash
./pg_porting_assistant
```

## Notes
For very long SQL queries, conversion may not be possible depending on the LLM model.

(When using OpenAI's API, the limit is approximately 1,500 lines (10,000 tokens) due to fast-agent configuration constraints.)

## Configuration
#### fastagent.config.yaml
A YAML-format configuration file for basic settings, which must be located in the same directory as the executable.  
If the configuration file is missing, a sample file is automatically generated when the executable is run.  
```yaml
default_model: openai.gpt-4o-mini             ## Default is OpenAI's model, requires OpenAI API key.

generic:
  base_url: "http://localhost:11434/v1"

logger:
  level: info                                 ## info / debug
  type: console                               ## file / console
  path: ./logs/pg_porting_assistant.log       ## Log location if type is file
  progress_display: true
  show_chat: true
  show_tools: true
  truncate_tools: true

mcp:
  servers:
    fetch:
      command: "uvx"
      args: ["mcp-server-fetch"]
    filesystem:
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-filesystem", "."]

sqlporter:
  models:                                      ## Agent model settings for converters
    converter_1: "generic.gemma3:4b"
    converter_2: "generic.llama3.2:1b"
    converter_3: "openai.gpt-4o-mini"

  instructions:                                ## Additional conversion rules for converters
    converter_1: ""
    converter_2: |
      Convert the Oracle SQL to PostgreSQL using Llama3-specific syntax rules.
      Respond ONLY with JSON as described in the default instruction.
    converter_3: ""

  settings:
    max_refinements: 3                         ## Maximum query refinement attempts
    min_rating: "EXCELLENT"                    ## Query quality [ EXCELLENT / GOOD ]
    retry_limit: 3                             ## Maximum retry attempts
    comment_prefix: "--"
```

#### fastagent.secrets.yaml
A configuration file for sensitive information such as API keys.  
It is recommended to set 600 permissions.  
```yaml
openai:
    api_key: sk-abcdefg ## Enter OpenAI API Key
mcp:
    servers:
        brave:
            env:
                BRAVE_API_KEY: <your_api_key_here>
```

#### ASIS Directory
The ASIS directory can have subdirectories and is where the original Oracle SQL files are located.  
It must be in the same location as the pg_porting_assistant executable, and the utility will attempt to convert all files in its subdirectories.  

#### TOBE Directory
The TOBE directory is where ported SQL files are stored, mimicking the subdirectory structure of ASIS.  
It must be in the same location as the pg_porting_assistant executable. If it doesn't exist, it will be created, and converted SQL files will be stored in its subdirectories.  

Notes  
1. All results require additional validation by the user.
2. Testing was conducted only in the RockyLinux 8 environment.
