## Phase 1: Preparation & Creating Reference Files

Before building a skill, you must define the ideal step-by-step process, identify necessary tools, and gather good output examples, as these have the biggest impact on performance.

**How to generate context files if you don't have them:** If you lack documentation about your brand, strategy, or target audience, you can have the AI build it for you.

1. Prompt Claude with a request like: "I want to create a [Topic] strategy document that I can later give when I'm building skills. Ask me some questions about my [Topic] strategy."
2. Use the **planning mode** to answer the AI's questions iteratively.
3. Once the AI has enough context, have it generate the strategy document to use as a reusable reference file for future skills.

Phase 2: Methods for Building a Skill

There are three concrete ways to create a custom skill:

- **Method 1: Manual to Automated.** Perform a specific task manually alongside your AI agent. Once completed successfully, simply ask the agent to **"save it as a skill."**
- **Method 2: Direct Prompting.** Instruct the AI to build the skill right away by giving it a prompt and uploading your reference files.
- **Method 3: Customizing Existing Skills.** Download a general skill from a marketplace and import it into your account. Navigate to **skills**, click **edit**, and ask Claude to combine the existing skill with your specific knowledge sources to create a highly tailored version.

Phase 3: The Skill Prompting Framework

When directly prompting Claude to build or update a skill, use this exact structural framework to get the best outcome:

1. **Define the Name and Trigger:** Explicitly state the skill name and the exact user phrasing that should activate it. _Example:_ "The name should be Infographic Generator and should be triggered anytime a user mentions he wants to generate or create an infographic."
2. **State the Goal:** Briefly define what the skill achieves and the style guidelines it must follow.
3. **Specify Connectors/MCPs (Model Context Protocols):** Tell the AI exactly which software, APIs, specific tables, or pages it needs to access to complete the task.
4. **Map the Step-by-Step Process (****skill.md****):** Lay out the exact workflow.
    - **Keep the core process file (****skill.md****) clean.** It should only contain the operational steps (like an SOP). Put all extra contextual information into separate reference files.
    - **Design the "Human in the Loop" UI:** Specify exactly when the AI should pause for your input. Instruct it to use dynamic QA boxes like **checkboxes, open fields, or single-select menus** to get your approval before moving forward.
5. **Define Expected Outputs:** Instead of accepting a single output, strictly instruct the AI to **generate multiple variations or options** at each step so you can choose the best one.
6. **Establish Rules & Guardrails:** Write down rules to prevent predictable errors. **Crucial tip:** Explicitly instruct the AI to use specific knowledge files as _obligatory steps_ in the process, otherwise, it tends to skip reading them.
7. **Enable Self-Learning:** Instruct the skill to automatically update itself. Add a rule stating that **if a user approves the final outcome, the skill must save that outcome as a "good example"** so it continuously trains itself on what success looks like.

Phase 4: Troubleshooting and Iteration Techniques

Skills require continuous iteration. When an output is flawed, use these specific commands to fix the underlying architecture:

- **If the AI doesn't follow the operational process correctly:** Ask it to make changes specifically to the **skill.md** file.
- **If the AI lacks information or context:** Ask it to create and add a new **reference file**.
- **If the AI makes a stylistic error you never want repeated** (e.g., using a black background): Instruct it to **add a rule** or update the knowledge file to explicitly forbid that action.
- **If the AI struggles to use a specific software tool (MCP):** First, guide the AI manually step-by-step to perform the necessary action in the software. Once successful, ask the AI to **create an "MCP reference doc"** that permanently records how to navigate and use that specific tool for the skill.

Phase 5: Exporting, Sharing, and Deployment Settings

Once your skill is perfected, you can distribute it to team members or marketplaces.

- **Export as a Zip:** Ask Claude to **give you a zip file** for your completed skill.
- **Installation:** Anyone else can install your skill by navigating to **Settings > Capabilities** and uploading that zip file into their own environment.
- **GitHub Deployment:** You can also ask Claude to bundle multiple skills into a "plugin" and deploy it directly through GitHub, which allows you to release version updates across any account using the plugin.