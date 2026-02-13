# Installation Guide

## How to Install This Custom Skill

### Option 1: Local Installation (Recommended for Development)

If you have the skill in a local directory:

```bash
# Navigate to where you saved the skill
cd /path/to/android-production-workflow-skill

# Install globally for all AI agents
npx skills add . -g -y
```

### Option 2: Install from GitHub

Once you push this skill to GitHub:

```bash
# Install from your GitHub repository
npx skills add yourusername/android-skills@android-production-workflow -g -y
```

**Repository structure should be:**
```
your-repo/
└── android-production-workflow/
    ├── SKILL.md
    ├── skill.json
    ├── README.md
    ├── docs/
    ├── examples/
    └── templates/
```

### Option 3: Install from Git URL

```bash
npx skills add https://github.com/yourusername/android-skills.git@android-production-workflow -g -y
```

## Verify Installation

Check that the skill was installed:

```bash
# List all installed skills
npx skills list

# You should see:
# android-production-workflow (1.0.0)
```

Check the skill location:

```bash
ls ~/.agents/skills/android-production-workflow
```

## Using the Skill

Once installed, your AI agents will automatically have access to:

1. **Complete architecture patterns** from `SKILL.md`
2. **Code templates** for quick scaffolding
3. **Working examples** for reference
4. **Best practices** for production apps

### Example Usage with AI Agent

**Prompt:** "Create a new MVVM feature for Products using the android-production-workflow skill"

**AI will:**
1. Use templates from the skill
2. Follow architecture patterns
3. Generate proper StateFlow ViewModels
4. Create Repository with caching
5. Build Compose UI screens

## Updating the Skill

If you modify the skill locally:

```bash
# Remove old version
npx skills remove android-production-workflow

# Reinstall updated version
npx skills add /path/to/android-production-workflow-skill -g -y
```

## Publishing to Skills Marketplace

To share with others:

1. **Create GitHub Repository:**
   ```bash
   cd android-production-workflow-skill
   git init
   git add .
   git commit -m "Initial commit: Android Production Workflow Skill"
   git remote add origin https://github.com/yourusername/android-skills.git
   git push -u origin main
   ```

2. **Tag a Release:**
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

3. **Submit to Skills.sh:**
   - Visit https://skills.sh
   - Submit your repository
   - Provide: `yourusername/android-skills@android-production-workflow`

## Troubleshooting

**"Skill not found" error:**
- Verify `skill.json` exists in root directory
- Check that `name` in `skill.json` matches install command
- Ensure proper directory structure

**AI not using the skill:**
- Reinstall with `-g` flag for global access
- Restart your AI agent/IDE
- Explicitly mention the skill in prompts

**Templates not working:**
- Verify templates directory exists
- Check template files have `.template` extension
- Ensure templates contain proper placeholder syntax

## Next Steps

1. **Read Documentation:**
   - Start with `README.md` for overview
   - Read `SKILL.md` for detailed patterns
   - Follow `docs/QuickStartGuide.md` for tutorial

2. **Try Examples:**
   - Study `examples/UserManagementExample.md`
   - Build a test project using the templates

3. **Customize:**
   - Modify templates to match your team's standards
   - Add new examples for common patterns
   - Extend documentation with team-specific guidelines

---

**Congratulations! Your custom Android Production Workflow skill is ready to use!** 🎉
