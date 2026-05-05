<div dir="rtl" align="right">

# News Comment Topic Analysis System

هذا الريبو يحتوي على كود ونوتبوكات مشروع فصلي لتحليل تعليقات الأخبار واستخراج المواضيع منها باستخدام نموذج embeddings محسّن مع BERTopic.

## نوع المشروع

Semester Project  
Syrian Private University  
Faculty of Artificial Intelligence Engineering

## فكرة المشروع

فكرة المشروع هي بناء نظام يستطيع تحليل عدد كبير من تعليقات الأخبار واكتشاف المواضيع المتكررة داخلها بشكل تلقائي.

المشروع ليس Supervised Classification، لأن البيانات لا تحتوي على labels جاهزة مثل politics أو health أو religion. لذلك لا نقيس النتيجة النهائية باستخدام accuracy أو precision أو recall كأنها classification task.

المشروع يعتمد على:

<span dir="ltr">Unsupervised Topic Modeling</span>

أما تدريب نموذج embeddings فتم باستخدام أزواج تعليقات مستخرجة تلقائيًا، ويمكن اعتباره نوعًا من:

<span dir="ltr">weakly-supervised / self-supervised fine-tuning</span>

لأننا لم نستخدم labels بشرية، بل بنينا أزواج <span dir="ltr">Comment ↔ Comment</span> اعتمادًا على وجود التعليقات في نفس المنشور والتشابه الدلالي بينها.

## خط العمل العام

<span dir="ltr">
Clean Comments → Final Fine-Tuned MiniLM Comment Encoder → Embeddings → UMAP → HDBSCAN → BERTopic Topics → Metrics + Manual Inspection
</span>

## النموذج النهائي المعتمد

اسم النموذج المعتمد في التقرير:

<span dir="ltr">Final Fine-Tuned MiniLM Comment Encoder</span>

الاسم الداخلي للتجربة:

<span dir="ltr">v7_fullft_minilm_raw_largepairs_1p5M_seq256</span>

النموذج الأساسي:

<span dir="ltr">sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2</span>

تم تدريب النموذج باستخدام:

- <span dir="ltr">1,500,000 Comment ↔ Comment positive pairs</span>
- <span dir="ltr">30,000 validation pairs</span>
- <span dir="ltr">MultipleNegativesRankingLoss</span>
- <span dir="ltr">Full fine-tuning</span>
- <span dir="ltr">max_seq_length = 256</span>

## النتائج النهائية

تم تقييم النموذج النهائي على مجموعة اختبار ثابتة تحتوي على:

<span dir="ltr">58,014 news comments</span>

| Metric | Value |
|---|---:|
| <span dir="ltr">Total Documents</span> | 58,014 |
| <span dir="ltr">Valid Topic Comments</span> | 39,458 |
| <span dir="ltr">Noise Comments</span> | 18,556 |
| <span dir="ltr">Valid Ratio</span> | 68.01% |
| <span dir="ltr">Noise Ratio</span> | 31.99% |
| <span dir="ltr">Detected Topics</span> | 94 |
| <span dir="ltr">Average Confidence Valid</span> | 82.52% |
| <span dir="ltr">Median Confidence Valid</span> | 98.08% |
| <span dir="ltr">Topic Diversity</span> | 0.9011 |

## لماذا تم اعتماد V7؟

تم اعتماد V7 لأنه أعطى أفضل نتيجة داخل BERTopic مقارنة بالنماذج السابقة. القرار لم يعتمد على training loss فقط، بل على نتائج downstream BERTopic مثل:

- تقليل <span dir="ltr">Noise Ratio</span>
- زيادة <span dir="ltr">Valid Topic Comments</span>
- عدد topics مضبوط
- ثقة أعلى في التعليقات المسندة إلى topics صحيحة
- الحفاظ على <span dir="ltr">Topic Diversity</span> جيدة

## التجارب الأخرى

### V8

تم تدريب V8 كتجربة طويلة من baseline model مع early stopping.  
رغم أنه أعطى نتائج أفضل قليلًا في بعض مقاييس التدريب والاسترجاع، لكنه كان أضعف داخل BERTopic. لذلك لم يتم اعتماده.

### V9 BGE

تمت تجربة نموذج <span dir="ltr">BAAI/bge-base-en-v1.5</span> كتجربة ثانوية لأن البيانات بالإنجليزية.  
رغم أنه أعطى sanity check جيد، إلا أن نتائج BERTopic كانت أضعف من V7. لذلك لم يتم اعتماده.

## بنية الريبو

```text
notebooks/
  01_pair_mining_v7_raw_largepairs.ipynb
  02_v7_final_fullft_minilm_training.ipynb
  03_v7_final_bertopic_evaluation.ipynb
  experiments/
    04_v8_baseline_long_training_rejected.ipynb
    05_v9_bge_secondary_experiment_rejected.ipynb
  06_streamlit_topic_inference_demo.ipynb

app/
  streamlit_app.py

results/
  final_metrics.csv
  final_topic_info.csv
  model_comparison.csv

docs/
  PROJECT_CONTEXT.md
  MODEL_HISTORY.md
  RESULTS_SUMMARY.md

report/
  final_report.html
