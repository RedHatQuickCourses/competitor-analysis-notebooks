# Competitor Analysis Use-case demonstration with Llamastack

> NOTE: This is still a work in progress. Some RAG, agent, and custom tool calling related features of Llamastack is demonstrated. We are still researching and implementing advanced agent workflows, and MCP server integration.

These notebooks are part of the "competitor analysis" use-case/prooof-of-concept focused on the Indian Banking industry.

Before running these notebooks on Red Hat OpenShift AI, you need to prepare the RHOAI environment. See https://github.com/rsriniva/competitor-analysis for instructions on how to prepare the environment.

The following are the notebooks and their purpose:

| Name | Description |
| :--- | :--- |
| 1-maas-test | Basic Testing of MaaS remote inference |
| 2-llamastack-test-basic | Basic Testing of Llamastack server set up |
| 3-simple-rag | Simple RAG chain with Milvus vector DB |
| 4-agentic-rag | Agentic RAG with Llamastack Agent API |
| 5-real-time-search| Llamastack websearch provider for real-time queries |
| 6-react-agent | Llamastack ReAct Agent with multi-tool workflow (Yahoo finance tool calling) |
