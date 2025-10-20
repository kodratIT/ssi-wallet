# 🤖 AI Quick Start - SSI Mobile Wallet

**Purpose**: Copy-paste prompts untuk AI assistant  
**Usage**: Copy salah satu prompt di bawah sesuai kebutuhan

---

## ⚡ PROMPT 1: Cek Status Project (Paling Sering Dipakai)

```
Saya mengerjakan rebuild project "SSI Mobile Wallet" - Self-Sovereign Identity wallet untuk iOS/Android.

Location: /Users/kodrat/Public/SSI/wallet

TASK: Cek status project saat ini

Tolong:
1. Baca file: .meta/PROGRESS-TRACKER.md
2. Report:
   - Overall progress (%)
   - Last completed story
   - Current/next task
   - Any blockers
3. Suggest apa yang harus dikerjakan next

Mulai dengan baca PROGRESS-TRACKER.md
```

---

## 🚀 PROMPT 2: Mulai Development Baru

```
PROJECT: SSI Mobile Wallet Rebuild
Location: /Users/kodrat/Public/SSI/wallet

CONTEXT:
- Tech: React Native 0.74 + Expo 51, TypeScript, Redux, Veramo
- Purpose: Self-Sovereign Identity wallet dengan W3C VC support
- Documentation: ROADMAP.md complete (10 phases, 112 stories)

TASK: Start implementation dari awal

Steps:
1. Read: ROADMAP.md (understand project)
2. Read: GET-STARTED.md (setup guide)
3. Check: .meta/PROGRESS-TRACKER.md (if exists)
4. Identify: Next phase/story to work on
5. Guide me: Step-by-step implementation

Let's begin!
```

---

## 🔄 PROMPT 3: Lanjutkan Development yang Ada

```
PROJECT: SSI Mobile Wallet
Location: /Users/kodrat/Public/SSI/wallet

CONTEXT:
Saya sudah mulai development tapi lupa sampai mana.

TASK:
1. Baca: .meta/PROGRESS-TRACKER.md (if exists)
2. Identify: Last completed phase/story
3. Verify: Apakah benar sudah complete (check files/code)
4. Tell me: What's next to do
5. Start: Next story implementation

Tolong mulai dengan cek progress tracker atau folder structure.
```

---

## 📝 PROMPT 4: Implement Phase/Story Spesifik

```
PROJECT: SSI Mobile Wallet
Location: /Users/kodrat/Public/SSI/wallet

TASK: Implement Phase [NUMBER] atau STORY-[NUMBER]

Steps:
1. Read: ROADMAP.md (understand phase objectives)
2. Read: phases/phase-[XX]-name/README.md (if exists)
3. Read: stories/STORY-[NUMBER]-description.md (if exists)
4. Summarize: What will be built
5. Confirm: Should we proceed?
6. Implement: Step by step
7. Verify: All acceptance criteria met
8. Update: Progress tracker

Tolong mulai dengan baca phase/story documentation.
```

*(Replace [NUMBER] dengan yang sesuai)*

---

## 🐛 PROMPT 5: Debug/Fix Error

```
PROJECT: SSI Mobile Wallet
Location: /Users/kodrat/Public/SSI/wallet

PROBLEM: [Describe error/issue]

CONTEXT:
- Working on: Phase [X] / STORY-[NUMBER]
- Error message: [paste error]
- Platform: iOS / Android / Both
- What I tried: [what you tried]

FILES TO CHECK:
1. Phase guide: phases/phase-XX/README.md
2. Story file: stories/STORY-XXX-*.md
3. Common errors section in documentation

TASK:
1. Analyze the error
2. Check if it's a known issue
3. Provide solution
4. Help me fix it step-by-step

Tolong bantu debug issue ini.
```

---

## 📊 PROMPT 6: Update Progress After Completion

```
PROJECT: SSI Mobile Wallet
Location: /Users/kodrat/Public/SSI/wallet

COMPLETED: Phase [X] / STORY-[NUMBER] - [Title]

STATUS: ✅ DONE

WHAT WAS DONE:
- [List what you did]
- [Files created/modified]
- [Commands run]
- [Tests passed]

VERIFICATION PASSED:
- [x] All acceptance criteria met
- [x] Code runs on iOS
- [x] Code runs on Android
- [x] No TypeScript errors
- [x] No ESLint warnings

TASK:
Update file .meta/PROGRESS-TRACKER.md:
- Mark Phase/Story as ✅ DONE
- Add completion date
- Add any notes/learnings

Then tell me what's next.
```

---

## 🎯 PROMPT 7: Create Phase/Story Documentation

```
PROJECT: SSI Mobile Wallet
Location: /Users/kodrat/Public/SSI/wallet

TASK: Create detailed documentation for Phase [NUMBER]

Context:
- Read: ROADMAP.md Phase [X] section
- Understand: Objectives, deliverables, timeline
- Reference: mobile-wallet project at /Users/kodrat/Public/SSI/mobile-wallet

Create:
1. phases/phase-[XX]-name/README.md
   - Detailed implementation guide
   - Step-by-step instructions
   - Architecture decisions
   - Common errors & solutions
   
2. Stories breakdown (STORY-XXX to STORY-YYY)
   - Task-by-task breakdown
   - Acceptance criteria
   - Code examples
   - Verification steps

Make it comprehensive and easy to follow.
```

---

## 📚 PROMPT 8: Understand SSI Concepts

```
PROJECT: SSI Mobile Wallet
Location: /Users/kodrat/Public/SSI/wallet

TASK: Explain SSI concepts needed for this project

Please explain:
1. What is Self-Sovereign Identity (SSI)?
2. What are Verifiable Credentials (W3C VC)?
3. What are DIDs (Decentralized Identifiers)?
4. How does OID4VCI (credential issuance) work?
5. How does SIOPv2/OID4VP (credential presentation) work?
6. What is Presentation Exchange (PEX)?

Context:
- I'm building an SSI wallet
- I need to understand these before coding
- Use simple explanations with examples

Make it beginner-friendly but technically accurate.
```

---

## 🔍 PROMPT 9: Analyze Existing Mobile Wallet Code

```
PROJECT: SSI Mobile Wallet (analyzing reference)
Location Reference: /Users/kodrat/Public/SSI/mobile-wallet
Location Target: /Users/kodrat/Public/SSI/wallet

TASK: Analyze how [FEATURE] is implemented in existing wallet

Feature to analyze: [e.g., "credential issuance flow", "DID creation", "QR scanning"]

Steps:
1. Find relevant files in mobile-wallet
2. Explain the architecture
3. Show key code snippets
4. Explain the flow
5. Identify dependencies
6. Suggest how to rebuild it

Help me understand so I can rebuild it better.
```

---

## 📋 PROMPT 10: Create Setup Scripts

```
PROJECT: SSI Mobile Wallet
Location: /Users/kodrat/Public/SSI/wallet

TASK: Create setup/utility scripts

Create scripts for:
1. Project initialization (setup.sh)
2. Dependency check (check-deps.sh)
3. Development start (dev.sh)
4. Build scripts (build-ios.sh, build-android.sh)
5. Test runner (test.sh)
6. Clean/reset script (clean.sh)

Requirements:
- Work on macOS
- Check prerequisites
- Give clear error messages
- Support both iOS and Android

Create practical, production-ready scripts.
```

---

## 🚨 PROMPT 11: Emergency - Project Stuck

```
PROJECT: SSI Mobile Wallet
Location: /Users/kodrat/Public/SSI/wallet

SITUATION: Project is stuck, need help ASAP

CONTEXT:
- Stuck on: [describe what you're stuck on]
- Last completed: [last phase/story]
- Error/blocker: [describe issue]
- What tried: [what you've tried]
- Time stuck: [how long]

TASK: Analyze and unblock

Steps:
1. Understand the blocker
2. Check if it's a known issue
3. Provide multiple solutions
4. Recommend best approach
5. Help implement the fix

URGENT - please help debug this ASAP!
```

---

## 🎓 PROMPT 12: Setup Development Environment

```
PROJECT: SSI Mobile Wallet
Location: /Users/kodrat/Public/SSI/wallet

TASK: Help me setup development environment

System Info:
- OS: macOS [your version]
- Node: [your version or "not installed"]
- Platform needed: iOS / Android / Both

Please help me:
1. Check what I have installed
2. Tell me what's missing
3. Guide me to install missing tools
4. Verify everything is working
5. Test with a "Hello World" React Native app

I'm following GET-STARTED.md but need hands-on help.
```

---

## 💡 Tips for Using These Prompts

### 1. Personalize the Prompt
- Replace `[NUMBER]` with actual phase/story number
- Replace `[describe...]` with your specific situation
- Add more context if needed

### 2. Give AI Time to Read
- AI needs to read files first
- Don't rush the response
- Let AI ask questions if unclear

### 3. Verify AI Understanding
- Check if AI read the right files
- Confirm AI understands the task
- Ask AI to summarize before implementing

### 4. Update Progress
- Always update .meta/PROGRESS-TRACKER.md
- Keep status current
- Document learnings

---

## 📂 Essential Files Reference

```
MUST READ FIRST:
ROADMAP.md                     ← Complete 10-phase plan
GET-STARTED.md                 ← Quick start guide
.meta/PROGRESS-TRACKER.md      ← Current status (when created)

IMPLEMENTATION:
phases/phase-XX/README.md      ← Phase details (when created)
stories/STORY-XXX-*.md         ← Task details (when created)

REFERENCE:
/Users/kodrat/Public/SSI/mobile-wallet  ← Existing wallet code

AI GUIDES:
AI-QUICK-START.md              ← This file
```

---

## 🎯 Most Common Use Cases

### Use Case 1: "Saya baru mulai project"
→ Use **PROMPT 2**: Mulai Development Baru

### Use Case 2: "Saya lupa progress nya"
→ Use **PROMPT 1**: Cek Status Project

### Use Case 3: "Lanjutin yang kemarin"
→ Use **PROMPT 3**: Lanjutkan Development

### Use Case 4: "Error, tolong fix"
→ Use **PROMPT 5**: Debug/Fix Error

### Use Case 5: "Udah selesai 1 story"
→ Use **PROMPT 6**: Update Progress

### Use Case 6: "Belum paham SSI"
→ Use **PROMPT 8**: Understand SSI Concepts

### Use Case 7: "Mau buat documentation"
→ Use **PROMPT 7**: Create Phase Documentation

---

## ⚡ One-Liner Prompts

For quick tasks:

```
Check status:
"Baca .meta/PROGRESS-TRACKER.md atau folder structure di /Users/kodrat/Public/SSI/wallet dan report current status"

Next task:
"Apa next phase/story yang harus dikerjakan untuk SSI wallet rebuild?"

Continue:
"Lanjutkan development SSI Mobile Wallet di /Users/kodrat/Public/SSI/wallet dari terakhir kali"

Understand feature:
"Analyze bagaimana [feature] diimplementasi di /Users/kodrat/Public/SSI/mobile-wallet"

Update progress:
"Update PROGRESS-TRACKER.md, mark Phase/Story-[X] as done"

Create docs:
"Create detailed documentation untuk Phase [X] berdasarkan ROADMAP.md"

Setup help:
"Help me setup React Native + Expo development environment untuk iOS/Android"
```

---

## 📊 Success Indicators

AI is working correctly if:
- ✅ Reads ROADMAP.md or PROGRESS-TRACKER.md first
- ✅ Reports current status clearly
- ✅ Provides step-by-step guidance
- ✅ References existing code when needed
- ✅ Asks for clarification when needed
- ✅ Updates progress tracking
- ✅ Verifies understanding before implementing

---

## 🔗 Quick Links

### Project Files
- `ROADMAP.md` - 10 phases, 112 stories
- `GET-STARTED.md` - Setup guide
- `.meta/PROGRESS-TRACKER.md` - Status tracking (when created)

### Reference Project
- `/Users/kodrat/Public/SSI/mobile-wallet` - Existing wallet code

### External Resources
- [Veramo Docs](https://veramo.io/docs/basics/introduction)
- [W3C VC Spec](https://www.w3.org/TR/vc-data-model/)
- [OpenID4VCI](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html)
- [React Native](https://reactnative.dev/)

---

## 🎯 AI Assistant Workflow

### Recommended Flow:

1. **Start** → Use PROMPT 1 or PROMPT 2
2. **Implement** → Use PROMPT 4
3. **Stuck** → Use PROMPT 5 or PROMPT 11
4. **Complete** → Use PROMPT 6
5. **Repeat** → Back to step 2

### For Documentation:
1. Use PROMPT 7 to create phase docs
2. Use PROMPT 7 to create stories
3. Use PROMPT 6 to update tracker

### For Learning:
1. Use PROMPT 8 for SSI concepts
2. Use PROMPT 9 to analyze existing code
3. Use PROMPT 12 for environment setup

---

## 📝 Example Usage

### Example 1: Starting Fresh

```
Developer: [Paste PROMPT 2]

AI: Reads ROADMAP.md, GET-STARTED.md
    Reports: Phase 0 needs to be started
    Suggests: Create Phase 0 documentation first
    
Developer: Ok, let's create Phase 0 docs

AI: [Creates comprehensive Phase 0 documentation]
```

### Example 2: Continuing Work

```
Developer: [Paste PROMPT 3]

AI: Checks .meta/PROGRESS-TRACKER.md
    Reports: Phase 0 completed, Story 8/10 done
    Next: STORY-009 - Create theme system
    
Developer: Let's do STORY-009

AI: [Guides through story implementation]
```

### Example 3: Debugging Error

```
Developer: [Paste PROMPT 5 with error details]

AI: Analyzes error
    Checks common issues
    Provides 3 solutions
    Recommends best approach
    
Developer: Let's try solution 2

AI: [Guides through fix step-by-step]
```

---

## 🚀 Advanced Prompts

### For Architects

```
PROJECT: SSI Mobile Wallet at /Users/kodrat/Public/SSI/wallet

TASK: Design architecture for [component/feature]

Requirements:
- Scalable
- Secure
- Maintainable
- Platform-agnostic (iOS + Android)

Consider:
- Existing patterns in reference wallet
- SSI best practices
- React Native constraints
- Performance implications

Provide:
1. Architecture diagram (text/mermaid)
2. Component breakdown
3. Data flow
4. Security considerations
5. Trade-offs

Think like a senior architect.
```

### For Code Review

```
PROJECT: SSI Mobile Wallet at /Users/kodrat/Public/SSI/wallet

TASK: Review my implementation of [feature]

Files:
[paste file paths or code]

Review for:
- TypeScript best practices
- React Native patterns
- Security issues
- Performance concerns
- Code maintainability
- Test coverage

Provide specific suggestions for improvement.
```

---

## 💡 Pro Tips

### 1. Context is King
- Always mention project location
- Reference specific files
- Include error messages completely
- Describe what you've tried

### 2. Break Down Complex Tasks
- Don't ask AI to build entire phase at once
- Use story-by-story approach
- Verify each step before proceeding

### 3. Learn, Don't Just Copy
- Ask AI to explain code
- Understand the "why"
- Request alternatives
- Build knowledge incrementally

### 4. Keep Track
- Update progress tracker regularly
- Document blockers immediately
- Note learnings in story files
- Keep git commits clean

---

**Quick Start Version**: 1.0  
**Last Updated**: 2024  
**Usage**: Copy any prompt above and paste to your AI assistant  
**Tip**: Add specific details for better results 🎯

---

## 🎉 Ready to Start?

1. **Read** ROADMAP.md completely
2. **Read** GET-STARTED.md completely
3. **Choose** a prompt from above
4. **Paste** to your AI assistant
5. **Follow** AI's guidance
6. **Update** progress as you go

**Let's build an amazing SSI wallet! 🚀**
