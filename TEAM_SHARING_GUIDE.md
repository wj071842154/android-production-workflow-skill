# Team Sharing Guide - Android Production Workflow Skill

This guide explains how to share and use the `android-production-workflow` skill across your development team to ensure consistent, high-quality Android code.

---

## 🎯 Why Share This Skill?

### Benefits for Your Team:

1. **Consistent Code Style** - Everyone follows the same architecture patterns
2. **Faster Onboarding** - New developers learn production patterns quickly
3. **Reduced Code Review Time** - Code follows established conventions
4. **Higher Code Quality** - Battle-tested patterns, fewer bugs
5. **AI-Assisted Development** - Team members can generate code with AI using the same patterns

---

## 📦 Installation for Team Members

### Option 1: From GitHub (Public Repository)

**Recommended for teams using this skill**

```bash
# Install globally (available to all AI agents)
npx skills add wj071842154/android-production-workflow-skill -g -y
```

**Verification:**
```bash
npx skills list
```

You should see `android-production-workflow` in the list.

### Option 2: From Local Clone

If you want to customize the skill for your team:

```bash
# Clone the repository
git clone https://github.com/wj071842154/android-production-workflow-skill.git

# Install from local directory
cd android-production-workflow-skill
npx skills add . -g -y
```

---

## 🤝 Team Workflow

### For Team Leads / Senior Developers:

1. **Share the Skill**
   - Send installation command to team
   - Add to team onboarding checklist
   - Include in project README

2. **Customize (Optional)**
   - Fork the repository
   - Modify patterns to match team preferences
   - Share the forked version with team

3. **Set Expectations**
   - Establish that all new code should follow skill patterns
   - Use skill as code review reference
   - Update skill as patterns evolve

### For Team Members:

1. **Install the Skill**
   ```bash
   npx skills add wj071842154/android-production-workflow-skill -g -y
   ```

2. **Learn the Patterns**
   - Read `SKILL.md` for architecture overview
   - Study `docs/QuickStartGuide.md` for step-by-step tutorial
   - Review `examples/UserManagementExample.md` for complete implementation

3. **Use with AI**
   - Prompt AI: "Using android-production-workflow skill, create..."
   - AI will generate code following team standards
   - Review and integrate into project

---

## 💻 Usage Scenarios

### Scenario 1: New Feature Development

**Developer Task**: Create a new "Products" feature

**With AI Assistance**:
```
Using android-production-workflow skill, create a Products feature:
- Product domain model (id, name, price, description, imageUrl, category)
- ProductRepository with offline-first caching
- ProductViewModel with StateFlow state management
- ProductListScreen and ProductDetailScreen with Compose UI
- Hilt dependency injection setup
```

**Result**: Complete, production-ready feature following team standards

### Scenario 2: Code Review

**Reviewer**: "This ViewModel doesn't follow our patterns"

**Solution**:
```
Using android-production-workflow skill, review this ViewModel
and refactor it to use StateFlow for UI state and SharedFlow for events.

[paste code here]
```

**Result**: Refactored code matching team standards

### Scenario 3: Architectural Decisions

**Team Discussion**: "How should we structure our repository layer?"

**Reference**: 
- Open `SKILL.md` → "Repository Pattern" section
- Point to offline-first caching strategy
- Use as authoritative reference for team decision

### Scenario 4: Onboarding New Developer

**New Developer**: "How do we handle state management?"

**Resources**:
1. Send link: `https://github.com/wj071842154/android-production-workflow-skill/blob/main/SKILL.md`
2. Point to StateFlow examples in skill
3. Have them generate sample feature using skill with AI
4. Review their code against skill patterns

---

## 📋 Team Standards Checklist

Use this checklist during code reviews to ensure consistency:

### Architecture
- [ ] Code follows Clean Architecture (Domain/Data/Presentation layers)
- [ ] Dependencies point inward (Data → Domain ← Presentation)
- [ ] Domain layer has no Android dependencies
- [ ] Use cases are single-responsibility

### State Management
- [ ] ViewModels use `StateFlow` for UI state (not `LiveData`)
- [ ] ViewModels use `SharedFlow` for one-time events
- [ ] UI collects state with `collectAsStateWithLifecycle()`
- [ ] UI state is represented as sealed interface/class

### Dependency Injection
- [ ] Application class annotated with `@HiltAndroidApp`
- [ ] Activities/Fragments annotated with `@AndroidEntryPoint`
- [ ] ViewModels annotated with `@HiltViewModel`
- [ ] Dependencies injected via constructor with `@Inject`

### Repository Pattern
- [ ] Repository interface in domain layer
- [ ] Repository implementation in data layer
- [ ] Returns `Flow<Result<T>>` for data streams
- [ ] Implements offline-first caching strategy

### UI Development
- [ ] 100% Jetpack Compose (no XML layouts)
- [ ] Material Design 3 components
- [ ] Proper loading/success/error state handling
- [ ] Reusable components extracted

### Code Quality
- [ ] No nullable types unless necessary
- [ ] Proper error handling (try-catch in repository)
- [ ] Meaningful variable names
- [ ] KDoc comments on public APIs

---

## 🔄 Keeping Skill Updated

### For Team Leads:

**Monitor for Updates**:
```bash
# Check if new version available
npx skills list

# Update to latest version
npx skills update android-production-workflow
```

**Notify Team**:
When skill is updated with new patterns, notify team via:
- Slack/Teams message
- Email with changelog
- Team meeting discussion

### For Team Members:

**Update Regularly**:
```bash
# Update skill to latest version
npx skills update android-production-workflow -y
```

---

## 🎓 Training Resources

### Self-Paced Learning Path:

**Week 1: Understanding Architecture**
- Read `SKILL.md` - Architecture section
- Study `example-project/README.md` - Complete example
- Goal: Understand 3-layer architecture

**Week 2: Hands-On Practice**
- Follow `docs/QuickStartGuide.md` - Build sample feature
- Complete `examples/UserManagementExample.md` - CRUD implementation
- Goal: Generate working code with AI assistance

**Week 3: Real Project Integration**
- Use skill to create feature for team project
- Submit for code review
- Goal: Apply patterns to production code

**Week 4: Mastery**
- Review others' code using skill as reference
- Contribute improvements to skill (if customized fork)
- Goal: Become team expert on patterns

### Team Workshops:

**Workshop 1: Architecture Deep Dive (2 hours)**
- Presentation on Clean Architecture
- Live coding: Build feature using skill patterns
- Q&A and discussion

**Workshop 2: AI-Assisted Development (1 hour)**
- Demo: Using skill with AI agents
- Practice: Each developer generates a feature
- Share results and best prompts

**Workshop 3: Code Review Best Practices (1 hour)**
- Review example PRs using skill as standard
- Discuss common violations and solutions
- Establish team-specific conventions

---

## 🛠️ Customization for Your Team

### Forking and Customizing:

If your team has specific preferences:

```bash
# 1. Fork the repository on GitHub
#    Visit: https://github.com/wj071842154/android-production-workflow-skill
#    Click "Fork"

# 2. Clone your fork
git clone https://github.com/YOUR_TEAM_ORG/android-production-workflow-skill.git

# 3. Customize patterns
cd android-production-workflow-skill
# Edit SKILL.md, templates, examples

# 4. Commit and push changes
git add .
git commit -m "Customize for team preferences"
git push origin main

# 5. Team installs from your fork
npx skills add YOUR_TEAM_ORG/android-production-workflow-skill -g -y
```

### Recommended Customizations:

- **Package naming**: Update templates with your company package
- **API patterns**: Add team-specific API conventions
- **Testing standards**: Add testing patterns to skill
- **Design system**: Integrate company design system in theme templates
- **Code style**: Add team linting rules and formatting

---

## 📞 Support and Questions

### Internal Team Support:

1. **Skill Champion**: Designate 1-2 developers as skill experts
2. **Team Channel**: Create Slack/Teams channel for skill questions
3. **Office Hours**: Weekly Q&A session for skill-related questions

### External Resources:

- **Skill Repository**: https://github.com/wj071842154/android-production-workflow-skill
- **Android Official Docs**: https://developer.android.com/topic/architecture
- **Jetpack Compose Docs**: https://developer.android.com/jetpack/compose
- **Material Design 3**: https://m3.material.io/

---

## 📊 Success Metrics

Track these metrics to measure skill adoption success:

### Code Quality Metrics:
- [ ] % of new code following architecture patterns
- [ ] Reduction in architecture-related code review comments
- [ ] Increase in code reusability
- [ ] Decrease in bugs related to state management

### Team Productivity Metrics:
- [ ] Time to onboard new developer
- [ ] Time to develop new feature
- [ ] Code review cycle time
- [ ] Developer satisfaction score

### Adoption Metrics:
- [ ] % of team members with skill installed
- [ ] # of features built using skill patterns
- [ ] # of AI-assisted generations using skill
- [ ] Team contribution to skill improvements

---

## 🎉 Success Stories Template

Document team successes with skill usage:

```markdown
### Feature: User Authentication (Developer: John)

**Before Skill**:
- 3 days to implement
- 5 code review iterations
- 2 bugs found in production

**After Skill**:
- 1 day to implement (using AI generation)
- 1 code review iteration (pattern-compliant)
- 0 bugs (followed tested patterns)

**Savings**: 2 days dev time, 4 review cycles, cleaner code
```

---

## ✅ Team Onboarding Checklist

Use this checklist for new team members:

```markdown
## Android Development Onboarding

- [ ] Install Skills CLI: `npm install -g @dotclaude/skills`
- [ ] Install android-production-workflow skill: `npx skills add wj071842154/android-production-workflow-skill -g -y`
- [ ] Read SKILL.md (Architecture overview)
- [ ] Complete QuickStartGuide.md tutorial
- [ ] Study UserManagementExample.md
- [ ] Build sample feature using skill with AI
- [ ] Submit first PR using skill patterns
- [ ] Pass code review checklist
- [ ] Attend team architecture workshop
- [ ] Become skill champion candidate (optional)
```

---

## 🚀 Getting Started (Quick Reference)

### Day 1: Install
```bash
npx skills add wj071842154/android-production-workflow-skill -g -y
```

### Day 2-3: Learn
- Read documentation
- Study examples
- Watch team demos

### Day 4-5: Practice
- Generate sample features with AI
- Get feedback from team

### Week 2+: Production
- Use for real project features
- Contribute improvements
- Help onboard others

---

**Questions? Contact your team's Skill Champion or open an issue on GitHub!**

**Ready to build consistent, high-quality Android apps as a team!** 🚀
