# Model Card: Mood Machine

## 1. Model Overview

**Model type:**  
I compared the rule-based model in `mood_analyzer.py` with the machine learning model in `ml_experiments.py`.

**Intended purpose:**  
The models classify short text messages as positive, negative, neutral, or mixed. This project is a learning experiment, not a reliable way to determine someone’s actual emotional state.

**How it works:**  
The rule-based model scores words using manually selected vocabulary and a simple negation rule. The ML model learns patterns from labeled posts using word counts and logistic regression.

## 2. Data

**Dataset description:**  
The dataset contains 15 posts with 15 matching labels. I started with six posts, added six examples containing different language styles, and later added three examples to investigate failures.

**Labeling process:**  
I assigned labels based on the intended meaning of each post. Positive and negative feelings together received the label “mixed.” Factual statements without clear emotion received “neutral.”

Some labels depend on context. “This is fine” could be neutral or sarcastic. “Great, another flat tire. Just what I needed” was labeled negative because its intended meaning is sarcastic frustration.

**Important characteristics:**  

- Slang, such as “fire” and “no cap.”
- Emojis, including 😊 and 🥲.
- Sarcasm and mixed feelings.
- Simple and longer negation, such as “not happy” and “not very happy.”

**Possible issues:**  
The dataset is very small and contains only English examples. It has four positive, five negative, two neutral, and four mixed posts. It does not represent the full range of language, dialects, or situations people use.

## 3. How the Rule-Based Model Works

**Scoring rules:**  
Preprocessing converts text to lowercase and separates words and punctuation into tokens. Recognized positive words add one point, and negative words subtract one point. A mood word’s score is reversed when the immediately preceding token is “not,” “never,” or “no.”

A score above zero produces “positive,” below zero produces “negative,” and zero produces “neutral.” The current implementation does not predict “mixed.” Emojis remain tokens but do not affect the score.

**Strengths:**  
The model is easy to inspect and behaves predictably on simple examples. It correctly classified “I am happy,” “Today was terrible,” and “I have class tomorrow.” It also handled “I am not happy about this.”

**Weaknesses:**  
It misses sarcasm, unfamiliar vocabulary, and negation separated from a mood word. Positive and negative words can cancel out, causing mixed feelings to be labeled neutral.

**Targeted improvement:**  
Initially, “That game was fire” was classified as neutral. I added “fire” to the positive vocabulary, and the prediction changed to positive. However, “The building is on fire” also became positive. This showed that fixing one use of a word can create errors in another context.

## 4. How the ML Model Works

**Features used:**  
The ML model uses bag-of-words features from `CountVectorizer`. These represent word counts rather than the full meaning or order of a sentence.

**Training data:**  
Logistic regression trains on `SAMPLE_POSTS` and `TRUE_LABELS`.

**Training behavior:**  
Training accuracy was 100% with both 12 and 15 posts. After adding three labeled examples and retraining, the model correctly classified those examples, including “The building is on fire.”

These examples were included in training, so the results do not demonstrate performance on unseen text.

**Strengths and weaknesses:**  
The ML model learned useful patterns without requiring a separate manual rule for every example. Before the final three additions, it correctly classified “I am not very happy” as negative and “I am happy but sad” as mixed.

However, it still misclassified the intended sarcasm in “I love getting stuck in traffic.” Before retraining on the building example, it also classified “The building is on fire” as positive. Its small training dataset makes overfitting a concern.

## 5. Evaluation

**Evaluation method:**  
I compared predictions with the human labels and tested additional sentences interactively.

| Dataset size | Rule-based accuracy | ML training accuracy |
|---|---:|---:|
| 12 posts | 7/12, about 58% | 100% |
| 15 posts | 7/15, about 47% | 100% |

The rule-based model’s number of correct predictions stayed at seven. Adding labeled posts did not automatically change its rules. The ML model retrained on the expanded dataset.

The ML score measures training accuracy because the program evaluates the same posts it trains on. I did not measure accuracy on a separate held-out test set.

**Correct predictions:**  

- Both models classified “That game was fire” as positive after the rule-based vocabulary change.
- The rule-based model classified “I am not happy about this” as negative because “not” reversed the score for “happy.”
- In interactive testing before the final dataset expansion, the ML model classified “I am happy but sad” as mixed.

**Incorrect predictions:**  

- Both models classified “I love getting stuck in traffic” as positive during interactive testing, missing the intended sarcasm.
- The rule-based model classified “I am not very happy” as positive because it only checks the immediately preceding token for negation.
- The rule-based model classified “Happy for my friend but sad they are moving away 🥲” as neutral. “Happy” and “sad” canceled out, and the emoji had no score.

## 6. Limitations

The dataset is too small to support broad claims about accuracy. Neither model reliably understands context or sarcasm.

The rule-based model depends on manually chosen words and cannot currently return “mixed.” Its negation handling only covers adjacent tokens.

The ML model depends on the examples and labels provided. Its perfect training accuracy does not establish reliable performance on new messages. Longer posts and languages other than English were not evaluated.

## 7. Ethical Considerations

Misclassifying distress as positive or neutral could cause someone to overlook a message that needs attention. These models should not be used to diagnose mental health conditions or make important decisions about people.

**Overt bias:**  
The model treated “That game was amazing” as positive but initially treated “That game was fire” as neutral. This suggests uneven coverage of standard wording and slang. People who use slang or language styles missing from the dataset may receive less accurate predictions. This small test identifies a coverage issue, but does not establish performance differences across demographic groups.

**Overt compliance:**  
The AI assistant initially suggested adding “fire” to the positive vocabulary, but also raised the context problem and asked me to test “The building is on fire.” That test exposed a new error. The suggestion needed testing rather than automatic acceptance.

**Privacy:**  
Analyzing real personal messages could expose sensitive information. Any real application would need consent and appropriate protection for those messages.

## 8. Ideas for Improvement

- Collect a larger, more varied labeled dataset.
- Evaluate the ML model on a separate test set.
- Track positive and negative signals separately to distinguish mixed feelings from neutral text.
- Improve negation handling for phrases such as “not very happy.”
- Handle slang in context instead of assigning every use the same meaning.
- Test emoji meanings carefully because they can vary by context.
- Update `explain()` to use the same negation logic as `score_text`, so explanations match predictions.
- Test more language styles and document differences in performance.