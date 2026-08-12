# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Câu hỏi ngoài phạm vi và bot từ chối ngắn gọn, nên gần như không dùng context chính sách. | Bot khẳng định sai điều kiện đổi trả/bảo hành, trạng thái thanh toán hoặc thông tin tài khoản mà context không hỗ trợ. | Chặn phát hành; kiểm tra grounding/prompt và bắt buộc trích đúng nguồn chính sách trước khi trả lời. |
| Answer Relevance | Khách hỏi mơ hồ như “đơn của tôi sao rồi?” và bot yêu cầu mã đơn hoặc làm rõ ý định. | Bot trả lời về sản phẩm khi khách hỏi hoàn tiền, hoặc bỏ qua yêu cầu khóa tài khoản/thanh toán. | Sửa intent routing, bổ sung câu hỏi làm rõ và test các ý định dễ nhầm. |
| Context Recall | Với câu hỏi ngoài phạm vi, không có evidence nội bộ phù hợp để retrieve là chấp nhận được. | Context thiếu điều khoản đổi trả, thời hạn bảo hành hoặc quy trình hoàn tiền cần để trả lời. | Đây là lỗi retrieval: cải thiện query, coverage tài liệu, chunking hoặc chỉ mục trước khi chỉnh generation. |
| Context Precision | Truy vấn rộng về danh mục sản phẩm có vài chunk liên quan yếu nhưng câu trả lời vẫn dựa vào chunk đúng. | Các chunk đầu đều nhiễu, làm bot chọn nhầm chính sách hoặc lộ hướng dẫn không liên quan đến tài khoản. | Đây là lỗi retrieval/ranking: thêm filter metadata, hybrid search hoặc reranker; kiểm tra top-k. |
| Completeness | Khách chỉ hỏi một ý nhỏ và bot trả lời đúng ý đó, dù reference đầy đủ hơn cho tình huống tổng quát. | Bot bỏ sót bước, điều kiện hoặc ngoại lệ quan trọng của đổi trả, bảo hành, thanh toán hay xác minh tài khoản. | Cải thiện generation: prompt checklist theo intent, tổng hợp evidence và kiểm thử các câu hỏi nhiều điều kiện. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Chuẩn bị cùng một tập question, rubric và hai câu trả lời A/B có chất lượng khác nhau. Condition 1 trình bày A trước rồi B; condition 2 hoán đổi thành B trước rồi A, còn toàn bộ input và rubric giữ nguyên. Chấm nhiều lần với thứ tự ngẫu nhiên, ghi score từng answer và chênh lệch A-B ở hai condition. Nếu một answer tăng điểm một cách nhất quán chỉ vì được đặt đầu, dù nội dung không đổi, judge có position bias; nếu chênh lệch gần như giữ nguyên thì chưa có bằng chứng rõ ràng về bias này.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> Rubric ưu tiên tính đúng, bám sát evidence chính sách, đủ ý cần thiết và khả năng giúp khách thực hiện bước tiếp theo. Nêu rõ độ dài không được thưởng: câu trả lời ngắn nhưng đúng và có hành động cụ thể có thể đạt điểm cao. Đồng thời trừ điểm khi lan man, lặp lại, thêm claim không được hỗ trợ hoặc che khuất điều kiện quan trọng; yêu cầu judge chấm từng tiêu chí độc lập trước khi tổng hợp.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> Human labels là chuẩn tham chiếu để đo tương quan giữa judge và đánh giá nghiệp vụ. Calibration giúp phát hiện lệch hệ thống, ví dụ judge quá dễ với câu trả lời dài hoặc quá khắt khe với câu từ chối đúng chính sách; từ đó điều chỉnh rubric, prompt và threshold. Chỉ khi mức đồng thuận đủ tốt, score tự động mới đáng tin để làm quality gate và theo dõi regression.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.90 | Nghiêm ngặt nhất vì sai chính sách đổi trả/bảo hành, thanh toán hay tài khoản có thể gây thiệt hại và mất niềm tin; câu trả lời phải được evidence hỗ trợ. |
| Answer Relevance | 0.80 | Bot phải giải quyết đúng intent hoặc yêu cầu làm rõ hợp lý; câu trả lời lệch chủ đề làm tăng ticket và thời gian xử lý. |
| Completeness | 0.80 | Cần nêu đủ điều kiện, bước thực hiện và ngoại lệ chính để khách tự hành động, nhưng cho phép khác biệt diễn đạt so với reference. |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> Dùng offline evaluation trước khi merge/release: chạy golden set gồm câu hỏi đổi trả, bảo hành, thanh toán và tài khoản để chặn regression có thể lặp lại. Dùng online evaluation sau khi triển khai để theo dõi traffic thật, tỷ lệ fallback/escalation, feedback khách hàng và các intent mới; ví dụ phát hiện truy vấn về một chương trình khuyến mãi vừa thay đổi. Dùng human review cho case rủi ro cao hoặc mơ hồ như tranh chấp thanh toán, yêu cầu liên quan dữ liệu cá nhân, chính sách chưa có evidence và mẫu lỗi mới. Ba lớp bổ sung nhau: offline phòng lỗi đã biết, online phát hiện drift thực tế, còn human review xác nhận các quyết định nhạy cảm và tạo nhãn để cải thiện hai lớp kia.

---

## Part 2 — Core Coding (14:45–15:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | | | | | | | | | |
| E02 | | | | | | | | | |
| E03 | | | | | | | | | |
| E04 | | | | | | | | | |
| E05 | | | | | | | | | |
| M01 | | | | | | | | | |
| M02 | | | | | | | | | |
| M03 | | | | | | | | | |
| M04 | | | | | | | | | |
| M05 | | | | | | | | | |
| M06 | | | | | | | | | |
| M07 | | | | | | | | | |
| H01 | | | | | | | | | |
| H02 | | | | | | | | | |
| H03 | | | | | | | | | |
| H04 | | | | | | | | | |
| H05 | | | | | | | | | |
| A01 | | | | | | | | | |
| A02 | | | | | | | | | |
| A03 | | | | | | | | | |

**Aggregate Report**

- Overall pass rate: ____%
- Avg Context Recall: ____
- Avg Context Precision: ____
- Avg Faithfulness: ____
- Avg Relevance: ____
- Avg Completeness: ____
- Failure type distribution: ____

**Ba cases có Overall Score thấp nhất**

1. ID: ____ | Score: ____ | Failure type: ____
2. ID: ____ | Score: ____ | Failure type: ____
3. ID: ____ | Score: ____ | Failure type: ____

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:*

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | | |
| 4 | | |
| 3 | | |
| 2 | | |
| 1 | | |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| | | |
| | | |
| | | |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +10)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
