## AnythingLLM Integration
The recommended approach for deploying composed prompts to AnythingLLM is through the Developer API. This creates a seamless, automated pipeline from modular prompt authoring to live workspace updates.
 
### How it works
 
- Compose prompts using the SQL workflow.
- Export to build directory (.build/anythingllm/)
- Deploy directly to AnythingLLM workspaces via API
 
The integration uses AnythingLLM's /api/workspace/{workspace_slug}/update endpoint to push your composed system prompts directly into the target workspace. This API-first approach is the best practice as it enables automation, version control, and eliminates manual file management.
 
### CLI Tool Usage
The CLI tool accepts a workspace slug and the path to your composed prompt:
```bash
deno run --allow-read --allow-net <script> \
           --slug=<workspace-slug> \
           --file=<prompt_file_pah> \
           --workspaces-url=<AnythingLLM-url>
```