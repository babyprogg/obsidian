---
type: note
status: active
created: 2026-01-15
updated: 2026-01-15
tags:
  - project
related:
  - "[[Unique Learner]]"
---

# QuickStart pipeline

## Context
[Why this note was created / what prompted it]
Записи пайплайна КуикСтарт фичи

## Content
[Main information, thoughts, or description]
v1

# 🧠 Learning Session Pipeline v0.1 (Core)

## 0. Принцип v1 (запомни)

> Одна сессия = один когнитивный выигрыш

Не «узнал», а **стал увереннее в одном месте**.

---

## 1️⃣ Input: минимальный контекст

Никаких анкет на 40 вопросов.

```json
{
"goal":"grow to mid frontend",
"current_activity":"pet project",
"stuck_on":"useEffect",
"time_available":20,
"energy":"medium"
}

```

💡 Почему так:

- нейродивергентные люди _теряют поток_ на лишних вопросах
- остальное ты вытащишь по ходу

---

## 2️⃣ Cognitive Guess (эвристика, не истина)

Ты **не определяешь тип**, ты делаешь предположение.

```json
{
"abstraction":"low",
"learning_bias":"example-first",
"attention_window":20
}

```

📌 В v1 это **жёстко захардкожено** или inferred из input.

Позже станет адаптивным.

---

## 3️⃣ Session Intent Resolver

Ответ на вопрос:

> Что сейчас даст максимальный эффект?

```json
{
"intent":"clarify",
"concept":"useEffect dependency array"
}

```

Правила v1:

- если `stuck_on` → это и есть концепт
- если `time < 15` → только практика
- если `energy = low` → минимум текста

---

## 4️⃣ Knowledge Atom Selector

Ты НЕ учишь весь useEffect.

```json
{
"atom":"why dependencies cause re-run",
"level":"practical"
}

```

📌 Один атом. Всегда.

---

## 5️⃣ Format Selector (очень важный шаг)

Жёсткая логика v1:

```
abstraction low → example
project context → guided task

```

```json
{
"format":"guided_example"
}

```

---

## 6️⃣ Content Generator (шаблонный, но умный)

Ты не генерируешь «объяснение».

Ты генерируешь **капсулу**.

```json
{
"hook":"почему useEffect стреляет дважды",
"micro_explanation":"коротко, без терминов",
"example":"todo app",
"task":"угадай, когда сработает",
"success_criteria":"ты можешь предсказать запуск"
}

```

⏱ Вся капсула = 10–15 минут.

---

## 7️⃣ Interaction Checkpoint

Один вопрос. Всегда.

```json
{
"question":"сработает ли эффект при изменении X?",
"expected":"yes / no"
}

```

---

## 8️⃣ Outcome Evaluation (очень грубо)

```json
{
"confidence":"low | medium | high"
}

```

📌 Даже кнопка “кажется понял” — уже достаточно.

---

## 9️⃣ Session Closure

Закрытие — критично.

```json
{
"summary":"ты понял, почему effect перезапускается",
"apply_now":"проверь свой компонент",
"next_atom":"stale closure"
}

```

12.01.26

# 🧠 Learning Pipeline v0.1 - Architecture & Roadmap

## 📐 Архитектура

### Принцип: One Atom Learning

```
Одна сессия = один когнитивный выигрыш
НЕ "узнал много" → "стал увереннее в ОДНОМ месте"

```

### Pipeline Stages

```
Input → Profile → Intent → Atom → Format → Content → Checkpoint → Outcome
  ↓       ↓        ↓        ↓       ↓         ↓          ↓          ↓
 5min   rules   rules    graph   rules    templates   1Q      confidence

```

---

## 🎯 v0.1 Реализация (Текущая)

### Что работает:

1. **Cognitive Profiler** (Rule-based)
    - Inference из минимального input
    - Attention window mapping
    - Learning bias detection
2. **Intent Resolver** (Hardcoded rules)
    - `stuck_on` → `clarify`
    - `time < 15` → `practice`
    - `energy low` → `debug`
3. **Atom Selector** (Static graph)
    - Hardcoded mapping: concept → atom
    - One atom per session enforcement
4. **Format Selector** (Strict rules)
    - `abstraction low` → `guided_example`
    - `project context` → `interactive_task`
5. **Content Generator** (Template-based)
    - Hook + Explanation + Example + Task + Checkpoint
    - Fixed structure, dynamic content

### Что НЕ работает (yet):

- ❌ AI-powered content generation
- ❌ Dynamic knowledge graph
- ❌ Adaptive profiling (no feedback loop)
- ❌ Session continuity tracking
- ❌ Real-time difficulty adjustment

---

## 🚀 Roadmap to v1.0

### Phase 1: AI Integration (2-3 weeks)

**Goal:** Replace templates with AI-generated content

### Step 1.1: Content Generator v2

```tsx
async generateContent(atom, format, profile) {
  // Use chatWithAI with strict prompt template
  const prompt = buildContentPrompt(atom, format, profile);
  const response = await chatWithAI([{role: 'user', content: prompt}]);
  return extractJSON<ContentCapsule>(response);
}

```

**Requirements:**

- Prompts for each format type
- Strict JSON output validation
- Fallback to templates if AI fails
- Token budget per capsule: ~1000 tokens

### Step 1.2: Example Quality Control

```tsx
// Validate generated examples
function validateExample(code: string): boolean {
  // Check: Valid syntax
  // Check: Relevant to atom
  // Check: Under 30 lines
  return score > 0.8;
}

```

---

### Phase 2: Knowledge Graph (3-4 weeks)

**Goal:** Dynamic atom relationships

### Knowledge Graph Structure

```tsx
interface KnowledgeNode {
  id: string;
  atom: string;
  prerequisites: string[];
  next_steps: string[];
  difficulty: number;
  concepts: string[];

  // Learning paths
  paths: {
    'example-first': string[];
    'theory-first': string[];
  };
}

```

### Graph Builder

```tsx
// Build from curriculum + user progress
class GraphBuilder {
  async buildGraph(domain: string): Promise<KnowledgeGraph> {
    // 1. Parse curriculum (e.g., React docs)
    // 2. Extract concepts & dependencies
    // 3. Create nodes & edges
    // 4. Add difficulty scores
  }

  async addUserProgress(graph, progress): Promise<void> {
    // Mark completed atoms
    // Calculate mastery scores
    // Suggest next atoms
  }
}

```

**Data Sources:**

- Official docs (React, TypeScript, etc.)
- Curated learning paths
- User completion data

---

### Phase 3: Adaptive Profiling (2-3 weeks)

**Goal:** Learn user's cognitive patterns over time

### Profiler v2

```tsx
class AdaptiveProfiler {
  private history: SessionHistory[];

  inferProfile(input, history): CognitiveProfile {
    const baseProfile = this.inferFromInput(input);
    const adjustments = this.analyzeHistory(history);

    return {
      abstraction: adjust(baseProfile.abstraction, adjustments.abstraction),
      learning_bias: adjust(baseProfile.learning_bias, adjustments.bias),
      attention_window: adjust(baseProfile.attention_window, adjustments.attention)
    };
  }

  private analyzeHistory(history): Adjustments {
    // Patterns:
    // - Skip rate by format → adjust format preference
    // - Completion time vs estimate → adjust attention window
    // - Confidence scores → adjust difficulty preference
  }
}

```

### Feedback Loop

```
Session → Outcome → Update Profile → Next Session

```

**Metrics to track:**

- Skip rate by format
- Time on task vs estimate
- Checkpoint success rate
- Self-reported confidence
- Return rate (days between sessions)

---

### Phase 4: Session Continuity (1-2 weeks)

**Goal:** Connect atoms across sessions

### Session Manager

```tsx
class SessionManager {
  async getNextSession(userId: string): Promise<SessionInput> {
    const lastSession = await this.getLastSession(userId);
    const profile = await this.getProfile(userId);
    const graph = await this.getKnowledgeGraph();

    // Determine next atom based on:
    // 1. Last session outcome
    // 2. User's current position in graph
    // 3. Time since last session
    // 4. Energy/time available today

    return this.buildRecommendation();
  }
}

```

### Spaced Repetition

```tsx
// Review atoms based on forgetting curve
function shouldReview(atom: CompletedAtom): boolean {
  const daysSince = daysSinceCompletion(atom);
  const intervals = [1, 3, 7, 14, 30]; // Spaced intervals

  return intervals.some(interval =>
    daysSince >= interval && daysSince < interval + 1
  );
}

```

---

## 🔧 Technical Debt & Improvements

### High Priority

1. **Error Handling**
    - Graceful degradation when AI fails
    - Fallback to templates
    - User-friendly error messages
2. **Performance**
    - Cache AI responses for common atoms
    - Preload next task while user completes current
    - Optimize bundle size
3. **Testing**
    - Unit tests for each stage
    - Integration tests for full pipeline
    - A/B testing framework

### Medium Priority

1. **Telemetry**
    - Log every pipeline decision
    - Track stage execution times
    - Monitor AI quality scores
2. **UI/UX**
    - Progress persistence across browser refresh
    - Offline mode with cached content
    - Mobile optimization
3. **Content Quality**
    - Peer review system for AI-generated content
    - User ratings for capsules
    - Automated quality checks

---

## 📊 Success Metrics

### v0.1 Goals (Current)

- ✅ Pipeline executes end-to-end
- ✅ Content generation works (templates)
- ✅ UI demonstrates all stages
- ⏳ User can complete one session

### v1.0 Goals (3 months)

- [ ] 80% of capsules AI-generated
- [ ] Knowledge graph covers 3+ domains
- [ ] Adaptive profiling improves over 5+ sessions
- [ ] Session continuity tracks 30-day learning paths
- [ ] 70% session completion rate
- [ ] 4.0+ average user satisfaction

### v2.0 Vision (6 months)

- Multi-domain learning (React → TypeScript → Backend)
- Collaborative learning (pair programming sessions)
- Content marketplace (community capsules)
- Enterprise features (team analytics)

---

## 🎓 Key Learnings & Principles

### Design Principles

1. **Minimal Viable Profiling**
    - Don't ask what you can infer
    - Progressive enhancement of profile
2. **One Atom Rule**
    - Never teach multiple concepts in one session
    - Depth over breadth
3. **Format Follows Function**
    - Format choice is cognitive, not aesthetic
    - Strict rules beat ML in v1
4. **Checkpoint-Driven**
    - Every session must have verifiable outcome
    - Binary success criteria

### Anti-patterns to Avoid

❌ **Information Overload**: Too many tasks in one session ❌ **Abstract First**: Teaching theory before examples for low-abstraction learners ❌ **One Size Fits All**: Same content for everyone ❌ **No Feedback Loop**: Not tracking what works

---

## 🛠️ Implementation Checklist

### Immediate (This Week)

- [x] Core pipeline implementation
- [x] Integration with existing system
- [x] Demo UI
- [ ] Save/load pipeline sessions
- [ ] Basic telemetry logging

### Short-term (Next 2 Weeks)

- [ ] AI content generation for 1 format
- [ ] Prompt engineering for capsules
- [ ] Quality validation for AI output
- [ ] Error handling & fallbacks

### Medium-term (Next Month)

- [ ] Knowledge graph v1 (hardcoded, 10 atoms)
- [ ] Adaptive profiling (track 3 metrics)
- [ ] Session continuity (basic sequencing)
- [ ] A/B testing framework

### Long-term (3 Months)

- [ ] Full AI-powered pipeline
- [ ] Dynamic knowledge graph
- [ ] Multi-session learning paths
- [ ] Production-ready system

---

## 📚 Resources & References

### Learning Science

- **Cognitive Load Theory**: Minimize extraneous load
- **Spaced Repetition**: Optimize review intervals
- **Interleaving**: Mix practice types
- **Retrieval Practice**: Active recall > passive review

### Technical

- **Prompt Engineering**: How to constrain AI output
- **Knowledge Graphs**: Neo4j, graph algorithms
- **Personalization**: Collaborative filtering, content-based
- **EdTech Best Practices**: Learning analytics, adaptive systems

---

## 🤝 Contributing

### How to Test the Pipeline

```tsx
// 1. Import the pipeline
import { LearningPipeline } from './learning-pipeline';

// 2. Create input
const input = {
  goal: 'learn React hooks',
  current_activity: 'pet project',
  stuck_on: 'useEffect',
  time_available: 20,
  energy: 'medium'
};

// 3. Run pipeline
const pipeline = new LearningPipeline();
const session = await pipeline.createSession(input);

// 4. Inspect results
console.log('Generated session:', session);

```

### Adding New Formats

```tsx
// In ContentGenerator class
private generateNewFormat(atom, profile, input): ContentCapsule {
  return {
    hook: '...',
    micro_explanation: '...',
    // ... your format-specific fields
    checkpoint: { question: '...', expected: '...' },
    duration_estimate: 10
  };
}

// Register in generate() method
const templates = {
  'new_format': this.generateNewFormat,
  // ... existing formats
};

```



## Key Takeaways
- 
- 
- 

## Links
- [[Unique Learner]] - проект в котором это разрабатывается
- [[]] - 

## Sources
- 

## Next Steps
[Optional: what to do with this information]
- 
- 

---
**Last Updated**: 2026-01-15