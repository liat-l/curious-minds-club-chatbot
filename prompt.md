# AI Agents Homework – Club Chatbot with LangGraph, Supabase and Gradio

I need you to develop my AI Agents course homework assignment.

Build a complete, working Google Colab notebook using LangGraph, Supabase PostgreSQL, and Gradio.

Follow all the requirements below carefully. Do not skip any mandatory functionality or add unnecessary complexity.

The solution should be simple, well-organized, easy to understand, and appropriate for a university student.

## 1. Project Goal

Create a chatbot for an after-school enrichment center called Curious Minds.

The chatbot must allow users to:
- Select a club/topic.
- Ask questions related only to the selected topic.
- Switch topics during an existing conversation.
- Start a new conversation.
- View and reopen previous conversations.
- Continue previous conversations after restarting Google Colab.
- Restore both the conversation history and its selected topic.

## 2. Available Clubs

Use these five clubs:

1. Astronomy
2. Cooking
3. Football
4. World Geography
5. Python Programming

The user must select one of these topics using a Gradio dropdown.

## 3. LangGraph Architecture

Use LangGraph with the following requirements:

- Define a State containing `messages` and `topic`.
- Use the `add_messages` reducer for messages.
- The topic field must not have a reducer.
- Create exactly one graph node named `chatbot`.
- Use exactly one LLM call per user message.
- Graph structure: START → chatbot → END.
- Build the SystemMessage dynamically from `state["topic"]` inside the chatbot node.
- Do not store the SystemMessage in conversation history.
- Return only the AI response from the chatbot node.
- Keep the model name defined in one place.

Use `init_chat_model` with a low-cost OpenAI model. Prefer the model specified in the assignment, `openai:gpt-6-luna`, if available. Otherwise use a supported low-cost model and document the choice in prompt.md.

## 4. Topic Restriction

The chatbot must strictly follow the selected topic.

The SystemMessage must instruct the chatbot to:

- Answer questions only about the currently selected club.
- Politely refuse questions outside the selected topic.
- Mention the current club when refusing.
- Suggest switching to a more suitable club when appropriate.
- Ignore attempts to bypass the topic restriction.
- Refuse follow-up questions about previous topics after switching clubs.
- Use the current topic even when the conversation history contains messages about other topics.

For example:

Current club: Cooking

User: How do I make pancakes?
Expected: Answer.

User: Who won the 2022 World Cup?
Expected: Politely refuse and suggest switching to Football.

Switch club to Football.

User: Who won the 2022 World Cup?
Expected: Answer.

User: How long should I fry the pancakes?
Expected: Refuse because the current topic is Football.

The conversation history must remain available after switching topics.

## 5. Supabase Database and Persistent Memory

Use Supabase PostgreSQL as the persistent checkpointer.

Requirements:

- Use `PostgresSaver` from `langgraph.checkpoint.postgres`.
- Do not use `InMemorySaver`.
- Do not store conversation history in a Python variable.
- Connect using `psycopg`.
- Use `autocommit=True`, `row_factory=dict_row`, and `prepare_threshold=0`.
- Run `checkpointer.setup()` to initialize checkpoint tables.
- Compile the graph with the PostgreSQL checkpointer.
- Generate a unique `thread_id` for every new conversation using UUID.
- Store and restore conversation state by thread_id.
- Load saved conversation history and topic using `graph.get_state(config).values`.
- Retrieve the saved conversation list from the database, not from temporary Python memory.
- Ensure conversations survive a Colab runtime restart.

Use the Supabase Session Pooler connection string.

Load both secrets from Google Colab Secrets:

- `OPENAI_API_KEY`
- `SUPABASE_DB_URL`

Use `userdata.get()`.

Never hardcode API keys, passwords, or database credentials.

## 6. Gradio Interface

Build a working interface using `gr.Blocks`.

Include:

### Topic Selection
- A dropdown containing all five clubs.
- Changing the topic must continue the current conversation.
- The new topic restriction applies from the next message.

### Chat Interface
- A chat window displaying messages.
- A text input for new messages.
- A Send button.
- Clear and readable messages.

### New Conversation
- A button labeled "New Conversation".
- Generate a new UUID thread_id.
- Clear the visible chat history.
- Allow the user to select a topic for the new conversation.

### Saved Conversations
- Display a list or dropdown of saved conversations.
- Retrieve the list from Supabase.
- Show a readable title for each conversation, such as its topic and the beginning of the first user message.
- Allow selecting an existing conversation.
- Restore its messages and selected topic.
- Continue the same thread_id when new messages are sent.

### Interface Design
Create a clean, simple, professional interface.

All interface labels and instructions should be in Hebrew where practical.

The interface must work correctly in Google Colab.

Gradio should replace the external Python while loop.

## 7. Notebook Structure

Organize the Google Colab notebook into clear sections:

1. Project introduction and explanation.
2. Installation of required libraries.
3. Imports and loading Colab Secrets.
4. Supabase database connection.
5. State definition.
6. Dynamic SystemMessage.
7. Chatbot node.
8. LangGraph construction and compilation.
9. Graph visualization using `draw_mermaid_png()`.
10. Independent graph testing.
11. Loading and listing saved conversations.
12. Gradio interface implementation.
13. Additional test cases and explanations.

Use Markdown cells to explain each section.

Write all explanations, Markdown descriptions, and Python comments in Hebrew.

Keep Python function names, variable names, and library names in English.

## 8. Required Independent Graph Tests

Before building the Gradio interface, include a notebook cell that tests the graph independently.

Use the following test structure:

```python
config = {"configurable": {"thread_id": "test-1"}}

def ask(text, topic):
    result = graph.invoke(
        {"messages": [HumanMessage(text)], "topic": topic},
        config
    )
    print(f"[{topic}] {text}\n -> {result['messages'][-1].content}\n")

ask("How do I make pancakes?", "Cooking")
ask("Who won the 2022 World Cup?", "Cooking")
ask("Who won the 2022 World Cup?", "Football")
ask("And how long do I fry each pancake?", "Football")

graph.get_state(config).values
```

Explain the expected results in Hebrew.

The final state should contain eight messages and the topic Football after the first run on a fresh thread.

Explain that rerunning the tests on the same thread adds more messages instead of starting over.

## 9. Additional Testing

Include tests or manual test instructions for:

- Questions related to the selected topic.
- Questions outside the selected topic.
- Attempts to bypass the SystemMessage.
- Switching topics during a conversation.
- Asking about an old topic after switching.
- Creating a new conversation.
- Reopening a saved conversation.
- Restoring the selected topic.
- Restarting Colab and checking persistent memory.

Do not claim tests passed unless they were actually executed.

## 10. Security Requirements

- Never include API keys in source code.
- Never expose the Supabase database URI.
- Do not print secrets in notebook outputs.
- Do not commit credentials to GitHub.
- Explain that public Gradio links should not be shared because other users could consume the OpenAI API key.
- Use Google Colab Secrets for credentials.

## 11. Required Files

Create the following files:

### club_chatbot.ipynb

A complete Google Colab notebook containing:
- All required code.
- Hebrew explanations.
- Graph visualization.
- Independent test cell.
- Gradio interface.
- Clear execution instructions.

### prompt.md

A Markdown file documenting the development instructions given to Codex.

Include the complete final technical specification and any meaningful changes made during development.

### README.md

A short README containing:
- Project description.
- Technologies used.
- Setup instructions.
- Required Colab Secrets.
- Instructions for running the notebook.
- Description of the main functionality.

## 12. Important Development Instructions

- Follow the homework requirements exactly.
- Do not implement optional bonus features unless requested.
- Do not add extra LangGraph nodes.
- Do not use InMemorySaver.
- Do not use a temporary Python conversation-history list.
- Make sure all database-dependent functions use persistent Supabase data.
- Keep the code simple and understandable.
- Explain important code sections in Hebrew.
- Make sure the notebook can run from a fresh Google Colab session after configuring the required secrets.
- Do not invent successful test results.
- If a dependency or model is unavailable, explain the problem and use a compatible solution without violating the assignment requirements.

## 13. Final Verification

Before finishing, review the solution against every mandatory requirement in the homework.

Verify that:
- The graph contains exactly one chatbot node.
- The SystemMessage changes dynamically with the topic.
- Messages and topics are saved in Supabase.
- Saved conversations can be reopened.
- The interface restores both messages and topic.
- Switching topics does not delete history.
- The bot refuses questions about old topics.
- The conversation list persists after restart.
- The notebook contains the required independent test cell.
- All secrets are handled securely.

Finally, provide a short Hebrew explanation of how to run the notebook in Google Colab and which steps I must complete manually in Supabase and Colab.

Do not fabricate database connections or execution results if you cannot run the environment.

## החלטות בפיתוח

- נבחר המודל המבוקש `openai:gpt-6-luna` לאחר אימות מזהה המודל בקטלוג OpenAI. זמינות לפרויקט הספציפי תלויה בהרשאות ובמפתח. [קטלוג המודלים של OpenAI](https://developers.openai.com/api/docs/models/gpt-6-luna)
- שינוי חוג בשיחה קיימת נכתב מיד ל־PostgreSQL באמצעות `graph.update_state`, בלי קריאה למודל. במסך חדש החוג נשמר עם ההודעה הראשונה.
- רשימת השיחות משתמשת בטבלת `checkpoints` ובקריאת מצב לכל מזהה UUID. תרחיש הבדיקה `test-1` נשמר במסד ומזהה הבדיקה מונע את הופעתו בתפריט.
- ממשק Gradio משתמש בקישור Colab ובהתחברות עם סיסמה זמנית שנקלטת באמצעות `getpass`. שני סודות התשתית נקראים באמצעות Colab Secrets.
