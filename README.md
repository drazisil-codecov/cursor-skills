# Cursor Skills

Personal collection of [Cursor Agent Skills](https://docs.cursor.com/advanced/agent-skills) for enhanced AI-assisted development workflows.

## What are Cursor Skills?

Skills are markdown files that teach Cursor's AI agent how to perform specific tasks following your preferred patterns and practices. They help the agent:

- Follow your team's conventions and workflows
- Apply domain-specific knowledge automatically
- Execute complex multi-step processes consistently
- Use your preferred tools and patterns

## Skills in this Repository

### 🎯 track-pr-test-plan

Ensures PR test plans and validation checklists don't get lost after merge by creating Linear tracking issues.

**When to use**: When a PR contains post-deployment validation steps, feature flag rollouts, or multi-environment testing that needs tracking.

**Features**:
- Extracts test plans from PR descriptions
- Creates Linear issues with proper structure (Background, Validation Steps, Success Criteria, Rollback Plan)
- Assigns to PR author automatically
- Adds to current sprint
- Uses same team as original Linear issue
- Links back to PR for traceability

[View skill →](track-pr-test-plan/SKILL.md)

## Installation

### Option 1: Copy to Personal Skills Directory (Recommended)

```bash
# Clone this repo
git clone https://github.com/drazisil-codecov/cursor-skills.git

# Copy skills to Cursor's personal skills directory
cp -r cursor-skills/track-pr-test-plan ~/.cursor/skills/

# Skills will be available across all your projects
```

### Option 2: Copy to Project Skills Directory

```bash
# In your project root
cp -r /path/to/cursor-skills/track-pr-test-plan .cursor/skills/

# Commit to share with team
git add .cursor/skills/
git commit -m "Add track-pr-test-plan skill"
```

## Usage

Once installed, skills are automatically discovered by Cursor's agent. The agent will apply them when appropriate based on the skill's description.

**Example**: When reviewing a PR with a test plan, simply ask:
- "How should I track this test plan?"
- "Make sure this validation doesn't get lost after merge"

The agent will automatically read the skill and follow the process.

## Creating Your Own Skills

See the [Cursor Skills documentation](https://docs.cursor.com/advanced/agent-skills) for guidance on creating skills.

**Quick tips**:
- Keep SKILL.md under 500 lines
- Use specific, trigger-rich descriptions
- Include concrete examples
- Write in third person
- Progressive disclosure for detailed content

## License

MIT License - feel free to use, modify, and share these skills!

## Contributing

Have improvements or new skills? Feel free to open an issue or PR!
