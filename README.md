# ChatGPT Conversation Data Across Four Countries

This repository contains ChatGPT conversation data with demographic information collected from participants in four countries: **Brasil**, **India**, **Nigeria**, and **Pakistan**. Each conversation has been classified along several dimensions using LLM-based classifiers adapted from the [How People Use ChatGPT (OpenAI)](https://cdn.openai.com/pdf/a253471f-8260-40c6-a2cc-aa93fe9f142e/economic-research-chatgpt-usage-paper.pdf) and the [The Anthropic Economic Index report: Economic Primitives](https://www-cdn.anthropic.com/096d94c1a91c6480806d8f24b2344c7e2a4bc666.pdf).

## Files

For each country, two CSV files are provided:

- `user_demographics_<Country>.csv` — one row per participant, containing basic demographic attributes.
- `user_conversations_classified_<Country>.csv` — one row per conversation, containing conversation timestamp and classification labels along multiple dimensions.

| Country  | # Users | # Conversations |
|----------|---------|-----------------|
| Brasil   | 246     | 40,067          |
| India    | 557     | 88,958          |
| Nigeria  | 243     | 44,114          |
| Pakistan | 206     | 29,451          |

Join key between the two files: `user_id`.

## Columns: `user_demographics_<Country>.csv`

| Column    | Description                               |
|-----------|-------------------------------------------|
| `user_id` | Anonymous participant identifier.         |
| `Age`     | Self-reported age (years).                |
| `Gender`  | Self-reported gender.                     |
| `Country` | Country of residence.                     |

## Columns: `user_conversations_classified_<Country>.csv`

### Identifiers
| Column                   | Description                                                             |
|--------------------------|-------------------------------------------------------------------------|
| `user_id`                | Anonymous participant identifier (joins to demographics file).          |
| `conversation_id`        | Unique conversation identifier (one row per conversation).              |
| `conversation_timestamp` | Unix timestamp (seconds) when the conversation was created.             |

### OpenAI-style topic classification
Derived by prompting `gpt-5-mini` with the conversation snippet and asking it to assign one of 24 fine-grained topic labels (e.g., `computer_programming`, `tutoring_or_teaching`, `personal_writing_or_communication`, `health_fitness_beauty_or_self_care`). These fine labels are grouped into 7 coarse themes following the OpenAI taxonomy.

| Column               | Description                                                                                              |
|----------------------|----------------------------------------------------------------------------------------------------------|
| `Topic_Label_Fine`   | Fine-grained conversation topic (24 categories).                                                         |
| `Topic_Label_Coarse` | Higher-level theme: `Practical Guidance`, `Seeking Information`, `Writing`, `Self-Expression`, `Multimedia`, `Technical Help`, or `Other/Unknown`. |
| `asking_doing_expressing` | `Asking` - seeking information or advice; `Doing` - requesting the model to perform a task whose output is produced primarily by the model (drafting, coding, extracting, rewriting); `Expressing` - messages that are neither asking for information nor requesting a task. |


### Anthropic economic primitives classification
Derived by prompting `gpt-5-mini` with classifiers adapted from the Anthropic Economic Index, which characterise the context and complexity of the interaction rather than the topic.

| Column                     | Description                                                                                                              |
|----------------------------|--------------------------------------------------------------------------------------------------------------------------|
| `work_coursework_personal` | Primary use case of the conversation. `Work` - professional use for tasks that are part of the user's job; `Coursework` - help the user complete coursework in educational contexts; `Personal` - any domain that is not work or coursework. |
| `Complete_task_alone`      | `Yes` - the user could have completed the task without the Assistant (even if it would have taken longer); `No` - the user could not have completed the task without the Assistant. |
| `Multitasking`             | `Yes` - the user worked on multiple tasks over the course of the conversation; `No` — the user worked on a single task throughout.                                                             |
| `User_Education`           | Integer in `0–20`: the estimated years of formal education needed to understand the **user's prompts** in the conversation.                                                                    |
| `AI_Education`             | Integer in `0–20`: the estimated years of formal education needed to understand the **Assistant's responses** in the conversation.                                                             |

### Unsupervised clustering topics
Conversations were embedded with `gemini-embedding-001` and clustered (PCA + KMeans) to discover country-specific topic clusters, which were then labeled and grouped into broader themes.

| Column                    | Description                                                                                |
|---------------------------|--------------------------------------------------------------------------------------------|
| `Clustering_Topic`        | Country-specific cluster label (e.g., `Online Earning`, `Mathematics`, `AI Image Generation`). |
| `Clustering_Topic_Theme`  | Higher-level theme grouping the cluster labels.                                            |

## Notes on missing values

- `Multitasking` and `Complete_task_alone` are `NaN` for conversations where the classifier could not confidently decide.
- `asking_doing_expressing` / `work_coursework_personal` are `NaN` for a small number of conversations whose output did not match a valid label.
- `Clustering_Topic` / `Clustering_Topic_Theme` are `NaN` for conversations outside the retained clusters (e.g., noise clusters dropped during labeling).

## Loading example

```python
import pandas as pd

country = "India"
convos = pd.read_csv(f"user_conversations_classified_{country}.csv")
users  = pd.read_csv(f"user_demographics_{country}.csv")

df = convos.merge(users, on="user_id")
```

## Paper citation

If you use this dataset, please cite:

```
How People Use ChatGPT: Conversation-Level Evidence from India, Nigeria, Brazil and Pakistan. 
by Shreyasi Roy Chowdhuri, Kiran Garimella. 
https://gvrkiran.github.io/content/How_people_use_ChatGPT.pdf
```
