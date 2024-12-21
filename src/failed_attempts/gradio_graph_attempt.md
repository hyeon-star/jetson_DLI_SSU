# **Gradio-Based Sensor Data Visualization - Failure Cases**

In `src/failed_attempts/gradio_graph_attempt.md`, the graph generation feature was added to the existing sensor data retrieval function for implementation.

Refer to the descriptions of the functions for each measurement sensor` moisture_sensor_info`, `light_sensor_info`, and `dht_sensor_info`, in the existing chatbot_ui.md.

This document summarizes failed code and its causes from attempts to generate and display graphs through the Gradio UI. By documenting these failure cases, we aim to identify clues for problem-solving and provide a reference for future improvements.

---


In this attempt, the existing Gradio chatbot was extended by adding the graph_from_file function to generate graphs based on CSV data and display them in the Gradio chat window.

- **New Feature**: The `graph_from_file` function samples sensor data to generate and save graphs.
- **Issue Encountered**: While attempting to display the graph image in the Gradio UI chat window, an issue arose where the image failed to render.

---

## **1. Installation and Execution**

**Installing Required Libraries**
```bash
pip install gradio pandas matplotlib openai
```
---

## **Executing the Script**

### **graph_from_file**

This function reads data from a CSV file and generates a graph after sampling the data.


**Code**:
```python

import pandas as pd
import matplotlib.pyplot as plt

def graph_from_file(file_path, x_column, y_columns, sample_step=27):
    try:
        # Read the CSV file
        data = pd.read_csv(file_path)
        
        # Sample the data: Skip rows by sample_step after the first row
        sampled_data = data.iloc[::sample_step, :].reset_index(drop=True)

        # Remove % and C symbols from numeric values and convert
        for column in y_columns:
            if '%' in sampled_data[column].iloc[0]:
                sampled_data[column] = sampled_data[column].str.rstrip('%').astype(int)
            elif 'C' in sampled_data[column].iloc[0]:
                sampled_data[column] = sampled_data[column].str.rstrip('C').astype(int)

        # Generate the graph
        plt.figure(figsize=(10, 6))
        for column in y_columns:
            plt.plot(sampled_data[x_column], sampled_data[column], label=column)
        
        plt.xlabel(x_column)
        plt.ylabel("Values")
        plt.title("Sampled Sensor Data Visualization")
        plt.xticks(rotation=45)
        plt.legend()
        plt.tight_layout()

        # Save the graph
        graph_path = "sensor_data_graph.png"
        plt.savefig(graph_path)
        plt.close()

        return {"graph_path": graph_path, "message": "Graph created successfully with sampled data."}
    except Exception as e:
        return {"error": str(e)}


```

---

## **3. Updated sensor_functions**

The `sensor_functions` with the `graph_from_file` function added are as follows.

```python
sensor_functions = [
    # 기존의 moisture_sensor_info, light_sensor_info, dht_sensor_info는 chatbot_ui.md 참고
    {
        "type": "function",
        "function": {
            "name": "graph_from_file",
            "description": "Generates a graph using data from a CSV file with specified x and y columns.",
            "parameters": {
                "type": "object",
                "properties": {
                    "file_path": {
                        "type": "string",
                        "description": "The path to the CSV file containing the sensor data."
                    },
                    "x_column": {
                        "type": "string",
                        "description": "The column name to use as the x-axis (e.g., 'Timestamp')."
                    },
                    "y_columns": {
                        "type": "array",
                        "items": {"type": "string"},
                        "description": "A list of column names to use as the y-axis (e.g., 'Soil Moisture (%)')."
                    },
                    "sample_step": {
                        "type": "integer",
                        "description": "The step size for sampling data points to reduce graph density. Default is 27.",
                        "default": 27
                    }
                },
                "required": ["file_path", "x_column", "y_columns"]
            }
        }
    }
]

```

---

## **4. Gradio UI**
The Gradio UI integrating user input and OpenAI, including the `graph_from_file` function, is as follows.

```python
import gradio as gr
import json
from openai import OpenAI

# User message processing function
def ask_openai(llm_model, messages, user_message, functions):
    client = OpenAI()
    proc_messages = messages.copy()

    if user_message:
        proc_messages.append({"role": "user", "content": user_message})

    response = client.chat.completions.create(
        model=llm_model, messages=proc_messages, tools=functions, tool_choice="auto"
    )
    response_message = response.choices[0].message
    tool_calls = response_message.tool_calls

    if tool_calls:
        available_functions = {
            "graph_from_file": graph_from_file  # Added function
        }

        proc_messages.append(response_message)

        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions.get(function_name)
            if function_to_call:
                try:
                    function_args = json.loads(tool_call.function.arguments)
                    function_response = function_to_call(**function_args)
                    proc_messages.append({
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": function_name,
                        "content": json.dumps(function_response),
                    })
                except Exception as e:
                    proc_messages.append({
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": function_name,
                        "content": json.dumps({"error": str(e)}),
                    })

        second_response = client.chat.completions.create(
            model=llm_model, messages=proc_messages
        )
        assistant_message = second_response.choices[0].message.content
    else:
        assistant_message = response_message.content

    proc_messages.append({"role": "assistant", "content": assistant_message})
    return proc_messages, assistant_message

# Gradio Interface
messages = []

def process(user_message, chat_history):
    proc_messages, ai_message = ask_openai("gpt-4o", messages, user_message, functions=sensor_functions)
    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    gr.Markdown("# Sensor Data Retrieval and Graph Generation")
    chatbot = gr.Chatbot(label="Sensor Data Retrieval and Visualization")
    user_textbox = gr.Textbox(label="Input")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
```

---

## **5. Execution Example**
### Input Example
```
"Generate a graph from the CSV file. Set the x-axis as Timestamp, and the y-axis as Soil Moisture (%) and Temperature (C)."
```


### **Output Result**
- Generated graph image: `sensor_data_graph.png`
- x-axis: `Timestamp`
- y-axis: `Soil Moisture (%)`, `Temperature (C)`

---

## **6. Analysis of the Cause of Failure**

### **Issue Observed**
- The Gradio server ran successfully, and the graph seemed to be generated, but the graph image was not displayed in the chat window.
- The server logs indicated that the graph image file was saved, but only text was returned in the Gradio chat window.

### **Suspected Causes**
1. **File Output Issue in Gradio**:
   - The Gradio Chatbot component is optimized for text responses.
   - To display an image file in the chat window, a separate gr.Image component needs to be used.

2. **Path Issue**:

   - The generated image file sensor_data_graph.png may have been saved with a relative path, making it inaccessible to the Gradio UI.

3. **Missing frpc File Issue**:
   - Failed to generate a shareable link, causing the locally hosted Gradio server to fail in connecting externally.
   - This was due to the missing frpc_linux_aarch64_v0.2 file, as mentioned in the message.

---

## **7. Attempts to Resolve**
- Attempt 1: Modified the code to use the gr.Image component to return the graph image.
- Attempt 2: Changed the image file save path to an absolute path.
- Attempt 3: Manually downloaded the frpc file and placed it in the specified directory.
  
Despite these efforts, the image display in the Gradio chat window still failed.

---

## **8. Purpose of Documenting Failed Code**

1. **Analyzing the Cause of the Problem**:
    To document the reasons for failure and avoid repeating the same mistakes.

2. **Future Improvements**:
   - Research ways to properly return images using `gr.Image` or other Gradio components.
   - Install the missing frpc file to restore the sharing functionality of the Gradio server.
   **Learning Resource**:
   Failed code serves as an important learning resource, offering clues for future debugging processes and solutions.

---

## **9. Next Step**
1. Review the appropriate methods for image output by referring to the Gradio documentation.
2. Install the missing frpc file to restore the sharing functionality.
3. Check for other file access permission issues or path settings and modify the code accordingly.

---





































