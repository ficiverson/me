---
title: "Building an AI-Powered newspaper article approval system with Human-in-the-Loop"
meta_title: "Building an AI-Powered newspaper article approval system with Human-in-the-Loop"
description: "Building an AI-Powered newspaper article approval system with Human-in-the-Loop"
date: 2025-11-04T05:00:00Z
image: "/images/blog/middleware/editor.png"
categories: ["AI","news"]
author: "Fernando Souto"
tags: ["human in the loop","middleware", "Langchain"]
draft: false
---

**Audio of the post:**

<audio controls>
  <source src="/audios/middleware_en.mp3" type="audio/mp3">
  Your browser does not support the audio element.
</audio>

In the fast-paced world of digital journalism, editors face the daunting task of reviewing hundreds of article submissions daily. While AI can draft articles quickly, human oversight remains crucial for maintaining editorial standards, ensuring factual accuracy, and preserving journalistic integrity. This is where **Human-in-the-Loop (HITL) middleware** becomes invaluable.

In this blog post, we'll explore how to build a newspaper article approval system that combines AI content generation with human editorial oversight, using LangChain and Azure OpenAI. We'll walk through the implementation, show code examples, and discuss important caveats.

## The problem: scale meets quality

Modern newsrooms receive an overwhelming volume of article submissions. Traditional manual review processes can't scale, but fully automated systems risk publishing inappropriate or inaccurate content. The solution? An intelligent workflow that:

1. **AI generates** draft articles based on topics or prompts
2. **Humans review** each article for quality, accuracy, and editorial fit
3. **Iterative refinement** allows collaborative improvement
4. **Final approval** ensures nothing publishes without human consent

## Project overview

The solution uses a custom Human-in-the-Loop middleware built on LangChain that allows seamless integration of human decision points into automated workflows. The system includes:

- **Azure OpenAI integration**: For article generation
- **Flexible middleware**: Customizable human input points
- **Workflow composition**: Chain multiple steps with approval gates
- **Iterative refinement**: Multiple rounds of human-AI collaboration

## Architecture: the core components

#### Step 1: Setup and Configuration

```python
from newspaper_article_example import create_newspaper_workflow
from config import get_azure_openai_llm
from langchain.schema.runnable import RunnableLambda

# Initialize the workflow manager
llm = get_azure_openai_llm()  # Get Azure OpenAI instance
workflow_manager = create_newspaper_workflow(llm)
workflow = workflow_manager.create_article_approval_workflow()

# The workflow is now ready to use
# Input: string (article topic)
# Output: string (final approval result)
```

#### Step 2: Article Generation (AI Step)

**What happens:**
1. Input: Topic string (e.g., "The best president of Real Club Deportivo A Coruña.")
2. AI generates article content
3. Output: Dictionary with article data

```python
def generate_article_draft(topic_and_details: str) -> Dict[str, Any]:
    """
    Step 1: AI generates initial article draft
    
    INPUT STATE:
    - topic_and_details: "The best president of Real Club Deportivo A Coruña."
    
    PROCESS:
    1. Create detailed prompt for AI
    2. Call Azure OpenAI API
    3. Receive generated article content
    4. Structure data for next step
    
    OUTPUT STATE:
    {
        "topic": "The best president of Real Club Deportivo A Coruña.",
        "article": "In recent months, major tech companies...",  # Generated content
        "status": "draft",      # Initial status
        "version": 1,           # Version tracking
        "editor_notes": []      # Empty list for future notes
    }
    """
    print(f"\n📰 Generating article draft about: {topic_and_details}")
    
    # Construct detailed prompt
    prompt = f"""Write a professional news article about: {topic_and_details}

Requirements:
- Objective and factual tone
- Include key facts and context
- Follow journalistic style (who, what, when, where, why)
- 500-800 words
- Suitable for publication in a reputable newspaper
- Include a compelling headline
- Structure with clear paragraphs and sections"""
    
    if self.llm:
        try:
            # API call to Azure OpenAI
            response = self.llm.invoke(prompt)
            
            # Structure the output data
            return {
                "topic": topic_and_details,
                "article": response.content,  # AI-generated article
                "status": "draft",            # Current workflow state
                "version": 1,                 # Track versions for revisions
                "editor_notes": []            # Store editor feedback
            }
        except Exception as e:
            print(f"❌ AI generation failed: {e}")
            # Return fallback data structure (same format)
            return {
                "topic": topic_and_details,
                "article": f"Draft article about {topic_and_details} (simulated)",
                "status": "draft",
                "version": 1,
                "editor_notes": []
            }
```

**Data Flow:**
```python
Input: "The best president of Real Club Deportivo A Coruña."
  ↓
[AI Processing]
  ↓
Output: {
  "topic": "The best president of Real Club Deportivo A Coruña.",
  "article": "...",
  "status": "draft",
  "version": 1,
  "editor_notes": []
}
```

#### Step 3: Quality check (automated step)

**What happens:**
1. Input: Article data dictionary
2. Automated checks (word count, quotes, etc.)
3. Output: Same dictionary with quality warnings added

```python
def quality_check(data: Dict[str, Any]) -> Dict[str, Any]:
    """
    Step 2: Automated quality checks
    
    INPUT STATE:
    {
        "topic": "...",
        "article": "In recent months...",
        "status": "draft",
        "version": 1,
        "editor_notes": []
    }
    
    PROCESS:
    1. Extract article content
    2. Run quality metrics:
       - Word count
       - Presence of quotes
       - Presence of numbers
       - Sentence count
    3. Generate warnings if issues found
    
    OUTPUT STATE:
    {
        "topic": "...",
        "article": "...",
        "status": "draft",
        "version": 1,
        "editor_notes": [],
        "quality_warnings": ["Article may need quotes from sources"]  # NEW
    }
    """
    article = data['article']
    
    # Calculate quality metrics
    checks = {
        'word_count': len(article.split()),
        'has_quotes': '"' in article or "'" in article,
        'has_numbers': any(char.isdigit() for char in article),
        'sentence_count': article.count('.') + article.count('!') + article.count('?')
    }
    
    # Generate warnings based on checks
    warnings = []
    if checks['word_count'] < 300:
        warnings.append("Article may be too short")
    if checks['word_count'] > 1500:
        warnings.append("Article may be too long")
    if not checks['has_quotes']:
        warnings.append("Article may need quotes from sources")
    if checks['sentence_count'] < 10:
        warnings.append("Article may need more sentences")
    
    # Add warnings to data (preserving all existing data)
    if warnings:
        data['quality_warnings'] = warnings
        print(f"\n⚠️ Quality Warnings: {', '.join(warnings)}")
    
    return data  # Same structure, with warnings added
```

**Data Flow:**
```python
Input: {article: "...", status: "draft", ...}
  ↓
[Quality Checks]
  ↓
Output: {article: "...", status: "draft", quality_warnings: [...], ...}
```

#### Step 4: Editor review (Human-in-the-Loop step)

**What happens:**
1. Input: Article data with quality warnings
2. **PAUSE**: Human editor reviews article
3. Human makes decision (approve/reject/revise)
4. Output: Updated data with editor decision

```python
def editor_review(data: Dict[str, Any]) -> Dict[str, Any]:
    """
    Step 3: Human editor reviews the article
    
    INPUT STATE:
    {
        "topic": "...",
        "article": "...",
        "status": "draft",
        "version": 1,
        "editor_notes": [],
        "quality_warnings": [...]
    }
    
    PROCESS:
    1. Display article and metadata to editor
    2. PAUSE: Wait for human input
    3. Process editor decision:
       - "approve" → status = "approved"
       - "reject" → status = "rejected"
       - "revise" → status = "needs_revision"
    4. Collect additional information (notes, feedback)
    
    OUTPUT STATE (if approved):
    {
        "topic": "...",
        "article": "...",
        "status": "approved",           # CHANGED
        "version": 1,
        "editor_notes": ["Approval notes: Looks good"],  # UPDATED
        "editor_decision": "approved",   # NEW
        "quality_warnings": [...]
    }
    
    OUTPUT STATE (if needs revision):
    {
        "topic": "...",
        "article": "...",
        "status": "needs_revision",     # CHANGED
        "version": 1,
        "editor_notes": ["Revision feedback: Add more statistics"],  # UPDATED
        "editor_decision": "revise",     # NEW
        "revision_feedback": "Add more statistics",  # NEW
        "quality_warnings": [...]
    }
    """
    # Display article for review
    print(f"\n{'='*60}")
    print(f"📝 ARTICLE FOR EDITORIAL REVIEW")
    print(f"{'='*60}")
    print(f"Topic: {data['topic']}")
    print(f"Version: {data['version']}")
    print(f"Word Count: {len(data['article'].split())}")
    
    if 'quality_warnings' in data:
        print(f"\n⚠️ Quality Warnings: {', '.join(data['quality_warnings'])}")
    
    # Show full article content
    print(f"\n{'='*60}")
    print("ARTICLE CONTENT:")
    print(f"{'='*60}")
    print(data['article'])
    print(f"{'='*60}")
    
    # HUMAN INPUT POINT - Execution pauses here
    print(f"\n🤖 Editorial Decision Required:")
    print("Options:")
    print("  - 'approve': Accept article for publication")
    print("  - 'reject': Reject article")
    print("  - 'revise': Request revisions with feedback")
    
    # This is where the workflow pauses and waits for human input
    decision = self.middleware.get_human_input("Editor decision: ").strip().lower()
    
    # Process decision and update state
    if decision == 'approve':
        data['status'] = 'approved'
        data['editor_decision'] = 'approved'
        # Optional: Collect additional notes
        notes = self.middleware.get_human_input(
            "Optional editor notes (or press Enter to skip): "
        ).strip()
        if notes:
            data['editor_notes'].append(f"Approval notes: {notes}")
    
    elif decision == 'reject':
        data['status'] = 'rejected'
        data['editor_decision'] = 'rejected'
        reason = self.middleware.get_human_input("Rejection reason: ").strip()
        data['rejection_reason'] = reason
        data['editor_notes'].append(f"Rejection reason: {reason}")
    
    else:  # revise
        data['status'] = 'needs_revision'
        data['editor_decision'] = 'revise'
        # Collect revision feedback
        feedback = self.middleware.get_human_input("Revision feedback: ").strip()
        data['revision_feedback'] = feedback
        data['editor_notes'].append(f"Revision feedback: {feedback}")
    
    return data  # Return updated state
```

**Data Flow (approval path):**
```python
Input: {article: "...", status: "draft", ...}
  ↓
[HUMAN REVIEW - PAUSES HERE]
  ↓ Editor types: "approve"
  ↓
Output: {article: "...", status: "approved", editor_decision: "approved", ...}
```

**Data Flow (revision path):**
```python
Input: {article: "...", status: "draft", ...}
  ↓
[HUMAN REVIEW - PAUSES HERE]
  ↓ Editor types: "revise"
  ↓ Editor provides: "Add more statistics"
  ↓
Output: {
  article: "...",
  status: "needs_revision",
  revision_feedback: "Add more statistics",
  ...
}
```

#### Step 5: AI revision (conditional step)

**What happens:**
1. Input: Article data (only processes if status = "needs_revision")
2. AI revises article based on feedback
3. Output: Updated article with new version number

```python
def ai_revision(data: Dict[str, Any]) -> Dict[str, Any]:
    """
    Step 4: AI revises article based on editor feedback
    
    INPUT STATE (if revision needed):
    {
        "topic": "...",
        "article": "Original article content...",
        "status": "needs_revision",
        "version": 1,
        "revision_feedback": "Add more statistics"
    }
    
    PROCESS:
    1. Check if revision is needed (status = "needs_revision")
    2. If not needed, return data unchanged
    3. If needed:
       - Create revision prompt with original article + feedback
       - Call AI to generate revised version
       - Update article content
       - Increment version number
    
    OUTPUT STATE:
    {
        "topic": "...",
        "article": "Revised article with statistics added...",  # UPDATED
        "status": "revised",     # CHANGED
        "version": 2,            # INCREMENTED
        "revision_feedback": "Add more statistics",
        "last_feedback": "Add more statistics"  # NEW
    }
    """
    # Conditional processing: only revise if needed
    if data['status'] != 'needs_revision':
        return data  # Pass through unchanged
    
    print(f"\n🤖 AI revising article based on editor feedback...")
    
    # Create comprehensive revision prompt
    revision_prompt = f"""Original article topic: {data['topic']}

Current article:
{data['article']}

Editor feedback: {data['revision_feedback']}

Please revise the article based on the feedback above. 
Maintain journalistic standards and incorporate the editor's suggestions.
Keep the same length and style."""
    
    if self.llm:
        try:
            # Call AI for revision
            response = self.llm.invoke(revision_prompt)
            
            # Update article content
            data['article'] = response.content
            data['version'] += 1  # Increment version
            data['status'] = 'revised'
            data['last_feedback'] = data['revision_feedback']
            
            print(f"✅ Article revised (Version {data['version']})")
        except Exception as e:
            print(f"❌ Revision failed: {e}")
            data['status'] = 'revision_failed'
    
    return data
```

**Data Flow:**
```python
Input: {status: "needs_revision", article: "...", revision_feedback: "..."}
  ↓
[Check: status == "needs_revision"?]
  ↓ YES
[AI Revision]
  ↓
Output: {status: "revised", article: "[REVISED]", version: 2, ...}
```

#### Step 6: Final approval (decision step)

**What happens:**
1. Input: Article data with final status
2. Process based on status (approved/rejected/revised)
3. Output: Final result string

```python
def final_approval(data: Dict[str, Any]) -> str:
    """
    Step 5: Final editorial approval
    
    INPUT STATE (if approved):
    {
        "topic": "...",
        "article": "...",
        "status": "approved",
        "version": 1,
        "editor_notes": [...]
    }
    
    INPUT STATE (if revised):
    {
        "topic": "...",
        "article": "[Revised content]",
        "status": "revised",
        "version": 2,
        "revision_feedback": "..."
    }
    
    PROCESS:
    - If approved: Return approval message with article
    - If rejected: Return rejection message
    - If revised: Show revised article and ask for final decision
    - If revision failed: Return error message
    
    OUTPUT: Final result string
    """
    if data['status'] == 'approved':
        # Direct approval path
        return f"""✅ ARTICLE APPROVED FOR PUBLICATION!

Topic: {data['topic']}
Version: {data['version']}
Editor Notes: {', '.join(data['editor_notes']) if data['editor_notes'] else 'None'}

{'='*60}
FINAL ARTICLE:
{'='*60}
{data['article']}
{'='*60}"""
    
    elif data['status'] == 'rejected':
        return f"""❌ ARTICLE REJECTED

Topic: {data['topic']}
Reason: {data.get('rejection_reason', 'Not specified')}
Editor Notes: {', '.join(data['editor_notes'])}"""
    
    elif data['status'] == 'revised':
        # Show revised article for final review
        print(f"\n{'='*60}")
        print(f"📝 REVISED ARTICLE FOR FINAL REVIEW")
        print(f"{'='*60}")
        print(f"Topic: {data['topic']}")
        print(f"Version: {data['version']}")
        print(f"Previous Feedback: {data.get('revision_feedback', 'N/A')}")
        print(f"\n{'='*60}")
        print("REVISED ARTICLE:")
        print(f"{'='*60}")
        print(data['article'])
        print(f"{'='*60}")
        
        # Another human input point
        final_decision = self.middleware.get_human_input(
            "Final decision (approve/reject): "
        ).strip().lower()
        
        if final_decision == 'approve':
            return f"""✅ ARTICLE APPROVED AFTER REVISION!

Topic: {data['topic']}
Final Version: {data['version']}

{'='*60}
FINAL ARTICLE:
{'='*60}
{data['article']}
{'='*60}"""
        else:
            reason = self.middleware.get_human_input("Final rejection reason: ").strip()
            return f"""❌ ARTICLE REJECTED AFTER REVISION

Topic: {data['topic']}
Reason: {reason}"""
    
    elif data['status'] == 'revision_failed':
        return f"""❌ ARTICLE REVISION FAILED

Topic: {data['topic']}
The AI was unable to revise the article. Please review manually."""
    
    return f"⏳ Article status: {data['status']}"
```

#### Complete Workflow Composition

Now let's see how all these steps are composed together:

```python
# Step 1: Create workflow steps
workflow = (
    RunnableLambda(generate_article_draft)    # Step 1: AI generates
    | RunnableLambda(quality_check)            # Step 2: Automated checks
    | RunnableLambda(editor_review)            # Step 3: Human review (PAUSES)
    | RunnableLambda(ai_revision)              # Step 4: Conditional AI revision
    | RunnableLambda(final_approval)           # Step 5: Final processing
)

# Step 2: Execute workflow
result = workflow.invoke("The best president of Real Club Deportivo A Coruña.")
```

**Complete Data Flow:**

```python
Input: "The best president of Real Club Deportivo A Coruña."
  ↓
[generate_article_draft]
  ↓ {topic: "...", article: "...", status: "draft", version: 1}
[quality_check]
  ↓ {topic: "...", article: "...", status: "draft", version: 1, quality_warnings: [...]}
[editor_review] ← HUMAN INPUT POINT #1
  ↓ Editor types: "revise"
  ↓ Editor provides: "Add more statistics"
  ↓ {topic: "...", article: "...", status: "needs_revision", revision_feedback: "..."}
[ai_revision]
  ↓ {topic: "...", article: "[REVISED]", status: "revised", version: 2}
[final_approval] ← HUMAN INPUT POINT #2
  ↓ Editor types: "approve"
  ↓
Output: "✅ ARTICLE APPROVED AFTER REVISION! ..."
```

## Important caveats and considerations

### Caveat 1: AI hallucination and fact-checking

**Problem**: AI-generated content may contain factual errors or hallucinations.

**Solution**: 
- Always require human fact-checking before publication
- Use AI for drafting, not final verification
- Implement source citation requirements

```python
def fact_check_reminder(data):
    """Remind editor to fact-check"""
    print("\n⚠️ IMPORTANT: Please verify all facts, figures, and claims before approving.")
    print("Check sources, dates, names, and statistics.")
    return data

# Add to workflow
workflow = (
    RunnableLambda(generate_article)
    | RunnableLambda(fact_check_reminder)
    | RunnableLambda(editor_review)
)
```

### Caveat 2: bias and editorial standards

**Problem**: AI models may reflect training biases or not match editorial style.

**Solution**:
- Provide clear style guidelines in prompts
- Human editors must enforce editorial standards
- Regularly review AI output for bias

```python
def generate_with_style_guide(topic):
    style_guide = """
    Editorial Guidelines:
    - Objective, third-person perspective
    - No opinion or speculation
    - Balanced representation of all sides
    - Verify all claims with sources
    - Follow AP Style guidelines
    """
    
    prompt = f"{style_guide}\n\nWrite an article about: {topic}"
    return llm.invoke(prompt)
```

### Caveat 3: cost and rate limiting

**Problem**: Azure OpenAI API calls can be expensive and have rate limits.

**Solution**:
- Implement caching for similar topics
- Set token limits appropriately
- Monitor API usage and costs

```python
import functools
from functools import lru_cache

@lru_cache(maxsize=100)
def cached_article_generation(topic_hash: str, prompt: str):
    """Cache article generations to reduce API calls"""
    # Only generate if not cached
    return llm.invoke(prompt)

# Use caching
def generate_article_smart(topic):
    topic_hash = hash(topic)
    prompt = f"Write article about: {topic}"
    return cached_article_generation(topic_hash, prompt)
```

### Caveat 4: security and privacy

**Problem**: Article content may contain sensitive information.

**Solution**:
- Never log full article content in production
- Implement access controls
- Ensure secure API connections

```python
import logging

# Don't log full articles
def safe_logging(data):
    logger.info(f"Article generated for topic: {data['topic']}")
    logger.info(f"Article length: {len(data['article'])} characters")
    # Don't log: logger.info(f"Article content: {data['article']}")  # NO!
    return data
```

### Caveat 5: workflow timeouts

**Problem**: Human approval steps can take indefinitely.

**Solution**:
- Implement timeout mechanisms
- Send reminders for pending approvals
- Auto-reject after timeout

```python
import time
from datetime import datetime, timedelta

def approval_with_timeout(data, timeout_hours=24):
    """Approval step with timeout"""
    start_time = datetime.now()
    timeout = timedelta(hours=timeout_hours)
    
    print(f"⏰ Approval timeout: {timeout_hours} hours")
    
    decision = input("👤 Editor decision: ").strip()
    
    # In production, use async/await with timeout
    # if datetime.now() - start_time > timeout:
    #     data['status'] = 'timeout_rejected'
    
    return data
```

### Caveat 6: version control

**Problem**: Multiple revisions can lead to confusion about which version is current.

**Solution**:
- Maintain clear version history
- Store all iterations
- Show version comparison

```python
def store_version_history(data):
    """Store article versions"""
    version_history = data.get('version_history', [])
    version_history.append({
        'version': data['version'],
        'article': data['article'],
        'timestamp': datetime.now().isoformat(),
        'editor_feedback': data.get('revision_feedback', '')
    })
    data['version_history'] = version_history
    return data
```

## Best practices summary

1. **Always verify AI output**: never publish without human review
2. **Implement quality gates**: automated checks before human review
3. **Maintain audit trails**: log all decisions and changes
4. **Set clear guidelines**: orovide AI with detailed editorial standards
5. **Monitor costs**: track API usage and optimize prompts
6. **Handle errors gracefully**: implement fallbacks for API failures
7. **Respect privacy**: don't log sensitive content
8. **Set timeouts**: prevent workflows from hanging indefinitely

## Conclusion

Building an AI-powered article approval system with human-in-the-loop provides the perfect balance between automation and editorial control. By leveraging LangChain's workflow composition and custom middleware, we can create flexible systems that:

- **Scale** to handle high volumes
- **Maintain quality** through human oversight
- **Enable collaboration** between AI and editors
- **Ensure accountability** through approval workflows

The key is understanding that AI is a tool to assist human editors, not replace them. With proper implementation, caveats management, and best practices, you can build robust systems that enhance editorial workflows while maintaining journalistic integrity.

Find here a full execution example in the console asking for "The best president of Real Club Deportivo de A Coruña":

```python
(venv) fernando.souto@192 middleware % python main.py --newspaper-article "The best president of Real Club Deportivo A coruña"
📰 Running Newspaper Article Approval Workflow:
==================================================
✅ Azure OpenAI LLM configured successfully!

📰 Generating article draft about: The best president of Real Club Deportivo A coruña

============================================================
📝 ARTICLE FOR EDITORIAL REVIEW
============================================================
Topic: The best president of Real Club Deportivo A coruña
Version: 1
Word Count: 685

============================================================
ARTICLE CONTENT:
============================================================
**Headline:**  
Augusto César Lendoiro: Architect of Deportivo La Coruña's Golden Era

**A Coruña, Spain —** Real Club Deportivo de La Coruña, one of Spain's historic football institutions, has seen a number of presidents since its founding in 1906. Yet, among them, Augusto César Lendoiro stands out as the most transformative and celebrated leader, credited with steering the club through its most successful period in history. His presidency, spanning from 1988 to 2014, not only elevated Deportivo onto the national and European stage but also left an indelible legacy in both the city of A Coruña and Spanish football as a whole.

**Who Was Augusto César Lendoiro?**

Born in A Coruña in 1946, Lendoiro began his career outside of football, working as a teacher and later as a politician. His entry into the world of sports administration started with local clubs, but his appointment as president of Deportivo in June 1988 marked a turning point for both himself and the club. At the time, Deportivo languished in Spain’s Segunda División and faced severe financial difficulties.

**A Presidency That Changed Everything**

Lendoiro’s presidency quickly became synonymous with ambition and innovation. He inherited a club on the brink of bankruptcy, but through a series of strategic measures—including renegotiating debts, fostering local support, and attracting new sponsors—he stabilized the club’s finances. His eye for talent and willingness to take calculated risks in the transfer market soon paid off.

In the early 1990s, Lendoiro brought in legendary players such as Bebeto, Mauro Silva, Djalminha, and Fran, the latter a local icon. Under his leadership, the club achieved promotion to La Liga in 1991, setting the stage for a decade of remarkable achievements.

**Deportivo’s Golden Age**

The 1990s and early 2000s are widely regarded as Deportivo’s "Golden Era." The club finished as La Liga runners-up in 1994 and 1995, narrowly missing out on the title in a dramatic 1994 finale. The pinnacle came in the 1999-2000 season when Deportivo, under coach Javier Irureta and Lendoiro’s stewardship, won their first and only La Liga championship. It was a historic achievement for a club from Galicia, traditionally overshadowed by Spain’s larger teams.

During this period, Deportivo also claimed two Copa del Rey titles (1995, 2002) and three Spanish Super Cups (1995, 2000, 2002). On the European front, the team became regular fixtures in the UEFA Champions League, reaching the semi-finals in the 2003-04 season after a memorable comeback against AC Milan.

**Leadership Style and Legacy**

Lendoiro’s management style combined pragmatism with bold vision. He was known for his negotiating skills, often securing high-profile signings for modest fees. His ability to foster a sense of unity among players, staff, and supporters was instrumental in building a competitive and resilient squad.

Beyond silverware, Lendoiro’s legacy includes modernizing the club’s infrastructure, most notably the Riazor Stadium, and investing in youth development. His presidency also cultivated a strong connection between the club and the city of A Coruña, turning Deportivo into a symbol of regional pride.

**Challenges and Departure**

The latter years of Lendoiro’s tenure were marked by financial challenges, as the club struggled to compete with the resources of Spain’s football giants. Relegation battles and mounting debts eventually led to his resignation in January 2014, ending a 26-year presidency.

Despite these difficulties, Lendoiro is widely credited with raising Deportivo’s profile and establishing the club as a respected force in Spanish and European football. His stewardship remains the benchmark against which subsequent presidents are measured.

**Why Lendoiro Is Considered the Best**

Though Deportivo has had a number of capable presidents, none matched the impact of Lendoiro. He not only delivered unprecedented sporting success but also transformed the club’s identity and ambitions. Football historians, local fans, and former players alike consistently cite his vision, resilience, and leadership as the reasons behind the club’s greatest achievements.

**Conclusion**

As Deportivo La Coruña continues to navigate the challenges of modern football, the legacy of Augusto César Lendoiro endures. His era is remembered as a time when the club dared to dream—and succeeded—against the odds. For supporters and observers, Lendoiro remains, objectively and enduringly, the best president in the club’s storied history.
============================================================

🤖 Editorial Decision Required:
Options:
  - 'approve': Accept article for publication
  - 'reject': Reject article
  - 'revise': Request revisions with feedback

🤖 Human Input Required:
📝 Prompt: Revision feedback: 
👤 Your input: I need at the end the links of the sources used to create the content

🤖 AI revising article based on editor feedback...
✅ Article revised (Version 2)

============================================================
📝 REVISED ARTICLE FOR FINAL REVIEW
============================================================
Topic: The best president of Real Club Deportivo A coruña
Version: 2
Previous Feedback: I need at the end the links of the sources used to create the content

============================================================
REVISED ARTICLE:
============================================================
**Headline:**  
Augusto César Lendoiro: Architect of Deportivo La Coruña's Golden Era

**A Coruña, Spain —** Real Club Deportivo de La Coruña, one of Spain's historic football institutions, has seen a number of presidents since its founding in 1906. Yet, among them, Augusto César Lendoiro stands out as the most transformative and celebrated leader, credited with steering the club through its most successful period in history. His presidency, spanning from 1988 to 2014, not only elevated Deportivo onto the national and European stage but also left an indelible legacy in both the city of A Coruña and Spanish football as a whole.

**Who Was Augusto César Lendoiro?**

Born in A Coruña in 1946, Lendoiro began his career outside of football, working as a teacher and later as a politician. His entry into the world of sports administration started with local clubs, but his appointment as president of Deportivo in June 1988 marked a turning point for both himself and the club. At the time, Deportivo languished in Spain’s Segunda División and faced severe financial difficulties.

**A Presidency That Changed Everything**

Lendoiro’s presidency quickly became synonymous with ambition and innovation. He inherited a club on the brink of bankruptcy, but through a series of strategic measures—including renegotiating debts, fostering local support, and attracting new sponsors—he stabilized the club’s finances. His eye for talent and willingness to take calculated risks in the transfer market soon paid off.

In the early 1990s, Lendoiro brought in legendary players such as Bebeto, Mauro Silva, Djalminha, and Fran, the latter a local icon. Under his leadership, the club achieved promotion to La Liga in 1991, setting the stage for a decade of remarkable achievements.

**Deportivo’s Golden Age**

The 1990s and early 2000s are widely regarded as Deportivo’s "Golden Era." The club finished as La Liga runners-up in 1994 and 1995, narrowly missing out on the title in a dramatic 1994 finale. The pinnacle came in the 1999-2000 season when Deportivo, under coach Javier Irureta and Lendoiro’s stewardship, won their first and only La Liga championship. It was a historic achievement for a club from Galicia, traditionally overshadowed by Spain’s larger teams.

During this period, Deportivo also claimed two Copa del Rey titles (1995, 2002) and three Spanish Super Cups (1995, 2000, 2002). On the European front, the team became regular fixtures in the UEFA Champions League, reaching the semi-finals in the 2003-04 season after a memorable comeback against AC Milan.

**Leadership Style and Legacy**

Lendoiro’s management style combined pragmatism with bold vision. He was known for his negotiating skills, often securing high-profile signings for modest fees. His ability to foster a sense of unity among players, staff, and supporters was instrumental in building a competitive and resilient squad.

Beyond silverware, Lendoiro’s legacy includes modernizing the club’s infrastructure, most notably the Riazor Stadium, and investing in youth development. His presidency also cultivated a strong connection between the club and the city of A Coruña, turning Deportivo into a symbol of regional pride.

**Challenges and Departure**

The latter years of Lendoiro’s tenure were marked by financial challenges, as the club struggled to compete with the resources of Spain’s football giants. Relegation battles and mounting debts eventually led to his resignation in January 2014, ending a 26-year presidency.

Despite these difficulties, Lendoiro is widely credited with raising Deportivo’s profile and establishing the club as a respected force in Spanish and European football. His stewardship remains the benchmark against which subsequent presidents are measured.

**Why Lendoiro Is Considered the Best**

Though Deportivo has had a number of capable presidents, none matched the impact of Lendoiro. He not only delivered unprecedented sporting success but also transformed the club’s identity and ambitions. Football historians, local fans, and former players alike consistently cite his vision, resilience, and leadership as the reasons behind the club’s greatest achievements.

**Conclusion**

As Deportivo La Coruña continues to navigate the challenges of modern football, the legacy of Augusto César Lendoiro endures. His era is remembered as a time when the club dared to dream—and succeeded—against the odds. For supporters and observers, Lendoiro remains, objectively and enduringly, the best president in the club’s storied history.

---

**Sources**  
- https://www.rcdeportivo.es/en/club/club-history  
- https://elpais.com/deportes/2014/01/21/actualidad/1390321519_809019.html  
- https://www.lavozdegalicia.es/not
============================================================

🤖 Human Input Required:
📝 Prompt: Final decision (approve/reject): 
👤 Your input: approve

✅ Result: ✅ ARTICLE APPROVED AFTER REVISION!

Topic: The best president of Real Club Deportivo A coruña
Final Version: 2

============================================================
FINAL ARTICLE:
============================================================
**Headline:**  
Augusto César Lendoiro: Architect of Deportivo La Coruña's Golden Era

**A Coruña, Spain —** Real Club Deportivo de La Coruña, one of Spain's historic football institutions, has seen a number of presidents since its founding in 1906. Yet, among them, Augusto César Lendoiro stands out as the most transformative and celebrated leader, credited with steering the club through its most successful period in history. His presidency, spanning from 1988 to 2014, not only elevated Deportivo onto the national and European stage but also left an indelible legacy in both the city of A Coruña and Spanish football as a whole.

**Who Was Augusto César Lendoiro?**

Born in A Coruña in 1946, Lendoiro began his career outside of football, working as a teacher and later as a politician. His entry into the world of sports administration started with local clubs, but his appointment as president of Deportivo in June 1988 marked a turning point for both himself and the club. At the time, Deportivo languished in Spain’s Segunda División and faced severe financial difficulties.

**A Presidency That Changed Everything**

Lendoiro’s presidency quickly became synonymous with ambition and innovation. He inherited a club on the brink of bankruptcy, but through a series of strategic measures—including renegotiating debts, fostering local support, and attracting new sponsors—he stabilized the club’s finances. His eye for talent and willingness to take calculated risks in the transfer market soon paid off.

In the early 1990s, Lendoiro brought in legendary players such as Bebeto, Mauro Silva, Djalminha, and Fran, the latter a local icon. Under his leadership, the club achieved promotion to La Liga in 1991, setting the stage for a decade of remarkable achievements.

**Deportivo’s Golden Age**

The 1990s and early 2000s are widely regarded as Deportivo’s "Golden Era." The club finished as La Liga runners-up in 1994 and 1995, narrowly missing out on the title in a dramatic 1994 finale. The pinnacle came in the 1999-2000 season when Deportivo, under coach Javier Irureta and Lendoiro’s stewardship, won their first and only La Liga championship. It was a historic achievement for a club from Galicia, traditionally overshadowed by Spain’s larger teams.

During this period, Deportivo also claimed two Copa del Rey titles (1995, 2002) and three Spanish Super Cups (1995, 2000, 2002). On the European front, the team became regular fixtures in the UEFA Champions League, reaching the semi-finals in the 2003-04 season after a memorable comeback against AC Milan.

**Leadership Style and Legacy**

Lendoiro’s management style combined pragmatism with bold vision. He was known for his negotiating skills, often securing high-profile signings for modest fees. His ability to foster a sense of unity among players, staff, and supporters was instrumental in building a competitive and resilient squad.

Beyond silverware, Lendoiro’s legacy includes modernizing the club’s infrastructure, most notably the Riazor Stadium, and investing in youth development. His presidency also cultivated a strong connection between the club and the city of A Coruña, turning Deportivo into a symbol of regional pride.

**Challenges and Departure**

The latter years of Lendoiro’s tenure were marked by financial challenges, as the club struggled to compete with the resources of Spain’s football giants. Relegation battles and mounting debts eventually led to his resignation in January 2014, ending a 26-year presidency.

Despite these difficulties, Lendoiro is widely credited with raising Deportivo’s profile and establishing the club as a respected force in Spanish and European football. His stewardship remains the benchmark against which subsequent presidents are measured.

**Why Lendoiro Is Considered the Best**

Though Deportivo has had a number of capable presidents, none matched the impact of Lendoiro. He not only delivered unprecedented sporting success but also transformed the club’s identity and ambitions. Football historians, local fans, and former players alike consistently cite his vision, resilience, and leadership as the reasons behind the club’s greatest achievements.

**Conclusion**

As Deportivo La Coruña continues to navigate the challenges of modern football, the legacy of Augusto César Lendoiro endures. His era is remembered as a time when the club dared to dream—and succeeded—against the odds. For supporters and observers, Lendoiro remains, objectively and enduringly, the best president in the club’s storied history.

---

**Sources**  
- https://www.rcdeportivo.es/en/club/club-history  
- https://elpais.com/deportes/2014/01/21/actualidad/1390321519_809019.html  
- https://www.lavozdegalicia.es/not
============================================================
(venv) fernando.souto@192 middleware % 
```


