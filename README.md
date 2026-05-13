# LAB-n8n-Node-Study-Guide

# LAB | n8n Node Study Guide

## Overview
Systematic analysis of n8n nodes across two workflows: a PDF-based RAG system and an AI agent chat.

## Files
| File | Description |
|------|-------------|
| `n8n_node_reference_table.md` | Reference table documenting 12 nodes — parameters, settings, JSON input/output, and transformations |
| `lab_summary.md` | Short summary: most useful nodes, node selection logic, top debugging tip |
| `screenshots/` | Node configuration and execution screenshots |

## Workflows Analyzed
- **Workflow 2** — PDF-based RAG system (On Form Submission → Default Data Loader → Text Splitter → Embeddings OpenAI → Pinecone Vector Store → Reranker Cohere)
- **Workflow 3** — AI agent chat (When Chat Message Received → AI Agent → OpenAI Chat Model / Simple Memory / SerpAPI)

## Notes
- Workflow 1 (Google Sheets) was  executed — yet not documented 
- Default Data Loader currently throws a pdf-parse v2 error; fix: `npm install pdf-parse@^1` on the host server