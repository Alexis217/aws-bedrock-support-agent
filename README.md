# Bedrock Customer Support AI Agent

An autonomous customer support agent built on **Amazon Bedrock AgentCore**, using **Strands Agents**, Model Context Protocol (**MCP**) tools, Bedrock Knowledge Bases (RAG), and persistent long-term memory.

---

## 🏗️ Architecture Overview

The agent leverages a serverless BYOC (Bring Your Own Code) runtime deployed via AWS Bedrock AgentCore:
* **Foundation Model:** Amazon Nova Lite (`global.amazon.nova-2-lite-v1:0`)
* **Agent Framework:** Strands Agents SDK
* **Tool Integrations:**
  * **MCP Gateway Client:** Order tracking and refund processing via streamable HTTP.
  * **Knowledge Base (RAG):** Amazon Bedrock Knowledge Base for catalog queries, policies, and loyalty tiers.
  * **Code Interpreter:** Sandboxed Python code execution for deterministic loyalty point and discount calculations.
  * **Browser Tool:** `AgentCoreBrowser` for automated web navigation and content extraction.
* **Memory Management:** Bedrock AgentCore Memory hooks for cross-session context recall and preference persistence.

---

## 📂 Project Structure

```text
.
├── Browser_Tool/
│   ├── prompt.txt
│   └── Test_6_Browser_Tool.png
├── Knowledge_Base/
│   ├── prompt.txt
│   └── Test_3_Knowledge_Base_(RAG).png
├── Long-Term_Memory/
│   ├── prompt.txt
│   ├── Test_4_Long-Term_Memory_Session_A.png
│   └── Test_4_Long-Term_Memory_Session_B.png
├── Loyalty_Discount_Calculation/
│   ├── prompt.txt
│   └── Test_5_Loyalty_Discount_Calculation.png
├── Order_Tracking/
│   ├── prompt.txt
│   └── Test_1_Order_Tracking.png
├── Refund_Processing/
│   ├── prompt.txt
│   └── Test_2_Refund_Processing.png
├── main.py
└── README.md

```

---

## 🧪 Verification & Test Scenarios

| Test Case | Scenario | Evidence |
| --- | --- | --- |
| **Test 1 — Order Tracking** | Retrieves status, tracking number, and carrier via MCP tool. | `Order_Tracking/Test_1_Order_Tracking.png` |
| **Test 2 — Refund Processing** | Initiates item refund and confirms SLA via MCP gateway. | `Refund_Processing/Test_2_Refund_Processing.png` |
| **Test 3 — Knowledge Base (RAG)** | Fetches tier perks from Bedrock Knowledge Base. | `Knowledge_Base/Test_3_Knowledge_Base_(RAG).png` |
| **Test 4 — Long-Term Memory** | Persists customer preferences across independent sessions. | `Long-Term_Memory/` (Session A & B) |
| **Test 5 — Discount Calculation** | Executes accurate arithmetic using AgentCore Code Interpreter. | `Loyalty_Discount_Calculation/Test_5_Loyalty_Discount_Calculation.png` |
| **Test 6 — Browser Tool** | Navigates to a remote web page to retrieve document metadata. | `Browser_Tool/Test_6_Browser_Tool.png` |

---

## 📝 Project Reflection

### 1. Specific Design Decision and Rationale

A key architectural decision was delegating the loyalty discount calculations to the AgentCore Code Interpreter instead of relying on the LLM's internal reasoning. Financial operations require deterministic precision. By executing generated Python code inside an isolated sandbox, the agent computes tier discounts, point redemption thresholds, and caps accurately without risking hallucinated arithmetic. Additionally, wrapping this execution inside a dedicated `@tool` with a deterministic fallback ensured graceful degradation if the sandbox connection timed out.

### 2. Concrete Challenge and Resolution

During integration testing with the `AgentCoreBrowser` tool, the agent encountered repeated validation failures. The foundation model generated session names containing underscores (such as `udacity_visit_page`), which violated the strict regex pattern enforced by the underlying AWS service (`^[a-z0-9-]+$`). Because the model failed to correct its arguments autonomously, it entered a loop describing the validation error in plain text instead of retrying the action. I resolved this by updating the system prompt with explicit constraints: defining a rigid instruction that enforces lowercase alphanumeric characters and hyphens only, alongside adding a clean delimiter in the prompt string. This guided the model to invoke the browser tool with compliant arguments on the first attempt.

### 3. Production Extensions

To scale this agent for a production enterprise environment, I would implement three critical improvements:

* **Observability and Tracing:** Integrate AWS OpenTelemetry with Amazon CloudWatch and Bedrock tracing to capture latency metrics, tool invocation success rates, token utilization, and end-to-end user session paths.
* **Granular Session and State Management:** Transition the ephemeral memory storage into Amazon DynamoDB with fine-grained TTL policies, coupling it with AgentCore Memory semantic strategies to isolate sensitive tenant data across enterprise accounts.
* **Safety Guardrails:** Implement Amazon Bedrock Guardrails to enforce PII masking (redacting credit cards and addresses), block prompt injection attacks, and restrict customer support outputs strictly to validated corporate policies before dispatching them to end users.
