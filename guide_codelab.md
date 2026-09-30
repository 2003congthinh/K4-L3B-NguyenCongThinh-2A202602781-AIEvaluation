---
day: "D14"
title: "K4 — Level 3B, Ngày 14: AI Evaluation & Benchmarking Pipeline"
description: "Hoàn thiện evaluation core, xây dựng golden dataset 20 QA và phân tích benchmark trên câu trả lời thật của trợ lý OrbitTech Store."
outcomes:
  - "Hoàn thiện năm Task bắt buộc và kiểm chứng bằng bộ unit tests của repo."
  - "Tạo golden dataset 20 QA có evidence từ đủ 10 tài liệu nguồn."
  - "Sinh actual answers, đo năm metrics và thiết kế rubric chấm điểm 1–5."
  - "Phân tích ba trường hợp bằng 5 Whys và đề xuất cách kiểm tra regression."
prerequisites:
  - "Có Python 3.11 trở lên và Git để làm việc trên repository cá nhân."
  - "Có OpenAI API key để chạy phần sinh câu trả lời thật."
requiredTools:
  - "Trình soạn thảo, terminal và tài khoản GitHub."
  - "Các thư viện trong requirements.txt của starter repo."
requiresSubmission: true
workMode: "individual"
---

# K4 — Level 3B, Ngày 14: AI Evaluation & Benchmarking Pipeline

Bạn sẽ tạo một pipeline đánh giá AI có thể chạy lại: code chấm điểm, bộ 20 câu hỏi có đáp án tham chiếu, kết quả benchmark và báo cáo phân tích lỗi. Hệ thống được đánh giá là trợ lý **OrbitTech Store Customer Support**. Bạn cần Python 3.11+, Git, trình soạn thảo và OpenAI API key; phần viết evaluation core chưa cần gọi API.

Nguồn bài học là [K4-L3B-AI-Evaluation](https://github.com/VinUni-AI20k/K4-L3B-AI-Evaluation). Quy định nộp bài, chấm điểm và làm bài nằm trong [SUBMISSION.md](https://github.com/VinUni-AI20k/K4-L3B-AI-Evaluation/blob/main/SUBMISSION.md), [RUBRIC.md](https://github.com/VinUni-AI20k/K4-L3B-AI-Evaluation/blob/main/RUBRIC.md) và [RULES.md](https://github.com/VinUni-AI20k/K4-L3B-AI-Evaluation/blob/main/RULES.md).

Lab thuộc **AICB-P1 · Phase 1 · Ngày 14 trong 15 · K4**
- Làm **cá nhân**, thời lượng **225 phút**.
- Buổi học bắt đầu từ **9:15 đến 13:00**;
- **12:00**: bắt đầu demo và Q&A.
- Các checkpoint dưới đây theo [CHECKPOINTS.md](https://github.com/VinUni-AI20k/K4-L3B-AI-Evaluation/blob/main/CHECKPOINTS.md).

## Chuẩn bị repository và xác nhận môi trường — CP0

Mở repo nguồn trên GitHub trước khi cài đặt. Cuối pha này, bạn có repository cá nhân và terminal chạy được bộ tests. Baseline là trạng thái starter để đối chiếu với những lần kiểm tra sau.

1. Trên GitHub, chọn **Fork** và đặt tên repository theo mẫu bên dưới. Thay phần họ tên bằng tên viết liền không dấu, PascalCase; điền MSSV của chính bạn. Mỗi học viên dùng một repository riêng.

   ```text
   K4-L3B-<HoVaTen>-<MSSV>-AIEvaluation
   ```

2. Sao chép URL clone của fork từ GitHub và clone về máy. Đặt tên thư mục làm việc là `K4-L3B-HoVaTen-MSSV`, thay hai thành phần cuối bằng thông tin của bạn. Mở thư mục này trong editor; terminal phải đứng tại nơi chứa `template.py`, `requirements.txt` và `golden_dataset.json`. Tên thư mục trên máy và tên repository GitHub có quy định riêng như trên.

3. Trong terminal, tạo môi trường theo hệ điều hành. Với macOS/Linux, kiểm tra Python từ 3.11 trở lên rồi chạy:

   ```bash
   python3 --version
   python3 -m venv .venv
   source .venv/bin/activate
   ```

   Với Windows PowerShell, dùng Python 3.11 đã cài; nếu máy có 3.12, thay `-3.11` bằng `-3.12`:

   ```powershell
   py -0p
   py -3.11 -m venv .venv
   .venv\Scripts\Activate.ps1
   ```

4. Sau khi kích hoạt môi trường, cài dependencies và chạy baseline từ thư mục gốc:

   ```bash
   python --version
   python -m pip install -r requirements.txt
   python -c "import openai, dotenv, pytest; print('Environment OK')"
   pytest tests/ -v
   ```

Bạn cần thấy `Environment OK` và tests được thu thập. Starter nguyên trạng có mốc dự kiến **42 failed** do TODO chưa hoàn thiện. Nếu gặp lỗi import hoặc collection, sửa môi trường theo [guide_lab.md](https://github.com/VinUni-AI20k/K4-L3B-AI-Evaluation/blob/main/guide_lab.md) trước khi viết code.

**Hoàn thành CP0:** editor mở đúng repo, Python đạt yêu cầu và tests chạy đến các lỗi của starter. Giữ terminal này cho các pha tiếp theo.

## Hoàn thiện dữ liệu đầu vào và cách tính Overall — CP1

Mở `exercises.md` cạnh `template.py`. Trước khi tính điểm, bạn cần phân biệt dữ liệu dùng làm đáp án tham chiếu với câu trả lời do hệ thống sinh ra. Nếu trộn hai loại này, điểm số có thể trông hợp lệ nhưng không còn đo hành vi thật của trợ lý. Pha này kết thúc khi hai data models nhận đúng dữ liệu và ba tests của `overall_score()` pass.

Trong Lab, **golden dataset** là bộ câu hỏi cùng đáp án và evidence do bạn biên soạn từ corpus. **Actual answer** là câu trả lời thật được lưu sau khi chạy trợ lý. `QAPair` giữ câu hỏi, đáp án tham chiếu và context; `EvalResult` giữ actual answer cùng các điểm đánh giá. Dữ liệu đi từ câu hỏi sang kết quả qua hai cấu trúc này, nên tên trường và signature phải khớp starter.

Ba **answer metrics** đo câu trả lời: Faithfulness, Relevance và Completeness. Hai **retrieval metrics** đo các đoạn tài liệu được lấy về: Context Recall và Context Precision. `overall_score()` chỉ lấy trung bình ba answer metrics. Hai retrieval metrics được giữ riêng để bạn chẩn đoán bước lấy tài liệu khi phân tích benchmark.

1. Trong **Part 1 — Warm-up** của `exercises.md`, điền Exercises 1.1–1.3. Ghi trường hợp điểm thấp cần điều tra, cách thử bias và đề xuất quality gate. Đây là lập luận của bạn; các ngưỡng bạn đề xuất trong worksheet không thay thế công thức đã quy định trong code.

2. Trong **Task 1** của `template.py`, hoàn thiện `QAPair` theo docstring: `question`, `expected_answer`, `context`, `metadata`, `retrieved_contexts`. Giữ default cho các trường tùy chọn. Với list và dict, dùng `field(default_factory=...)` như gợi ý để mỗi instance có dữ liệu riêng. Kiểm tra rằng `context` là chuỗi gold evidence, còn `retrieved_contexts` là danh sách các chunks theo thứ tự truy xuất.

3. Hoàn thiện `EvalResult` và `overall_score()`. Giữ `qa_pair`, `actual_answer`, ba answer scores, `passed`, `failure_type` cùng hai retrieval scores tùy chọn. `None` ở một retrieval score có nghĩa là metric chưa được tính; giá trị `0.0` là kết quả đã tính. Đừng gộp hai trạng thái này khi viết model hoặc report.

4. Lưu code, đồng bộ file được tests sử dụng rồi chạy nhóm test của pha. Trong Lab này, chọn `template.py` làm bản làm việc chính. Sau mỗi nhóm TODO đã sửa, copy sang `solution/solution.py` trước khi chạy tests.

   - MacOS/Linux:

   ```bash
   cp template.py solution/solution.py
   pytest tests/test_solution.py::TestEvalResultOverallScore -v
   ```

   - Windows PowerShell:

   ```powershell
   Copy-Item template.py solution/solution.py
   pytest tests/test_solution.py::TestEvalResultOverallScore -v
   ```

Tests ưu tiên `solution/solution.py` khi file tồn tại; `evaluate_answers.py` lại import `template.py`. Vì vậy, lưu file trong editor chưa đủ để tests đọc bản mới. Nếu trước đó bạn đã làm trực tiếp trong `solution/solution.py`, đối chiếu và đưa phần hoàn thiện về `template.py` trước khi dùng lệnh copy; tránh ghi đè bài đã làm.

**Hoàn thành CP1:** nhóm `TestEvalResultOverallScore` đạt **3 passed**. Nếu lỗi xuất hiện ngay khi tạo `QAPair` hoặc `EvalResult`, đọc lại tên trường và defaults trước khi sửa phép tính. Khi models đã nhận đúng dữ liệu, bạn có thể triển khai từng metric ở pha tiếp theo mà không phải thay đổi cấu trúc đầu vào.

Khi toàn suite vẫn còn tests fail ở CP1, hãy xem chúng thuộc Task nào. Các Task 2–5 chưa hoàn thiện sẽ tiếp tục báo lỗi; dùng nhóm test của pha để quyết định chuyển bước.

## Tính năm metrics và kiểm tra LLMJudge — CP2

Tiếp tục trong `template.py`, tại **Task 2** và **Task 3**. Đích của pha là một evaluator trả được năm metrics đúng trách nhiệm và một judge xử lý được phản hồi theo interface đã cho. Các metrics trong starter dùng **word overlap**, tức mức giao nhau giữa các tập từ; kết quả cần được đọc cùng evidence khi bạn đánh giá chất lượng câu trả lời.

Hàm `_tokenize()` đã xử lý chữ thường, dấu câu và loại `STOPWORDS`. Hãy dùng hàm này trong các phép đo để cùng một văn bản được xử lý nhất quán. Điểm overlap không tự xác nhận một chính sách đúng về ý nghĩa: câu trả lời có thể dùng nhiều từ giống nguồn nhưng sai điều kiện. Khi báo cáo, bạn vẫn cần mở actual answer và đoạn evidence tương ứng.

### Mỗi metric đang so sánh những dữ liệu nào?

Trong benchmark của repo, adapter ghép gold evidence thành `QAPair.context`. Faithfulness dùng context này để so với actual answer. Context Recall và Context Precision dùng các chunks thật trong `retrieved_contexts`. Giữ đúng đường đi này khi triển khai; nó quyết định bạn có thể diễn giải mỗi score đến đâu.

| Metric | Dữ liệu so sánh | Điều cần kiểm tra |
|---|---|---|
| Faithfulness | Answer với gold context | Tỷ lệ từ của answer có trong context |
| Relevance | Answer với question | Tỷ lệ từ của question được answer phủ |
| Completeness | Answer với expected answer | Tỷ lệ từ của expected được answer phủ |
| Context Recall | Hợp các chunks với expected | Evidence cần thiết có được lấy về |
| Context Precision | Chunks theo hạng với expected | Chunks liên quan có đứng sớm |

**Average Precision@K** xét vị trí của các chunks liên quan. Theo docstring, một chunk được xem là liên quan khi phủ ít nhất `relevance_threshold`, mặc định `0.1`, của tập từ expected. Bạn tính precision tại từng hạng có chunk liên quan, rồi lấy trung bình trên số chunk liên quan. Chỉ đếm số chunks liên quan sẽ bỏ mất thông tin thứ tự mà metric này cần đo.

1. Hoàn thiện ba answer metrics theo docstring. Kiểm tra mẫu số của từng metric trước khi viết phép chia. Giữ điểm trong `[0.0, 1.0]` và xử lý trường hợp tập từ ở mẫu số rỗng theo contract: answer rỗng với Faithfulness, question rỗng với Relevance, expected rỗng với Completeness trả `1.0`. Khi một test fail, kiểm tra tập từ thực tế sau `_tokenize()` trước khi đổi công thức.

2. Hoàn thiện hai retrieval metrics. Context Recall dùng **hợp** tập từ của tất cả chunks, nên việc đổi thứ tự cùng một tập chunks không đổi coverage. Context Precision phải giữ thứ tự để tính điểm theo hạng. Expected rỗng trả `1.0`; với expected có nội dung, precision trả `0.0` khi không có chunks hoặc không có chunk liên quan.

3. Nối các phép đo trong `run_full_eval()`. Khi `contexts is None`, giữ hai retrieval scores là `None`; khi được truyền danh sách, kể cả danh sách rỗng, gọi các hàm retrieval. `passed` chỉ là `True` khi cả ba answer scores từ `0.5` trở lên. Giữ thứ tự phân loại đầu tiên khớp: Faithfulness dưới `0.3` → `hallucination`; Relevance dưới `0.3` → `irrelevant`; Completeness dưới `0.3` → `incomplete`; còn lại nếu failed → `off_topic`.

4. Trong `LLMJudge`, lưu callable `judge_llm_fn`, tạo prompt chứa question, answer và rubric, rồi đọc JSON scores từ kết quả. Theo interface hiện tại, scores trong code nằm trên thang `0–1`; nếu không parse được JSON scores, trả fallback `0.5` cho mỗi tiêu chí. Phần rubric bạn sẽ viết ở Exercise 3.3 dùng thang `1–5`. Giữ rõ hai thang này khi trình bày và không tự thay đổi contract của class. `detect_bias()` cần trả các khóa `positional_bias`, `leniency_bias`, `severity_bias`; hai ngưỡng trong docstring là trung bình `> 0.8` và `< 0.3`.

5. Đồng bộ hai file theo CP1 rồi chạy các nhóm tests sau từ thư mục gốc. Unit tests cung cấp callable phản hồi cho judge nên chưa cần API key.

   ```bash
   pytest tests/test_solution.py::TestRAGASEvaluator tests/test_solution.py::TestContextMetrics tests/test_solution.py::TestRetrievalMetricWiring::test_run_full_eval_connects_optional_retrieval_metrics -v
   pytest tests/test_solution.py::TestLLMJudge -v
   pytest tests/ -v
   ```

**Hoàn thành CP2:**
- Nhóm evaluator đạt **14 passed, 1 skipped**, nhóm judge **4 passed**;
- Toàn suite cộng dồn dự kiến **21 passed, 20 failed, 1 skipped**.
- Test skipped thuộc bonus reranking.
- Những tests còn fail cần thuộc phần Runner và Analyzer chưa hoàn thiện.
- Nếu retrieval score bị `None` khi test đã truyền chunks, kiểm tra nhánh `contexts` trong `run_full_eval()`
- Nếu điểm khác kỳ vọng, xác định đúng mẫu số và thứ tự hạng trước khi sửa.

Judge trong code phát hiện các tín hiệu bias nêu trong docstring. Worksheet còn yêu cầu lập phương án kiểm soát position, verbosity và self-preference bias; các yêu cầu đó cần phần lập luận riêng của bạn. Khi evaluator và judge qua checkpoint, chuyển sang nối pipeline để mỗi QA tạo một kết quả có thể truy ngược về câu hỏi ban đầu.

## Nối benchmark và công cụ phân tích lỗi — CP3

Mở **Task 4–5** trong `template.py`. Các metrics đã tính được từng score; bây giờ bạn cần chạy chúng trên danh sách QA, tổng hợp kết quả và nhận diện thay đổi giữa hai lần đánh giá. Cuối pha này, toàn bộ required tests phải pass để benchmark thật có thể dùng evaluation core của bạn.

`BenchmarkRunner.run()` nhận `qa_pairs`, một `agent_fn` nhận câu hỏi và trả câu trả lời, cùng evaluator. Với benchmark thật, adapter sẽ cấp một hàm trả lại actual answer đã lưu. Nhờ đó bạn có thể sửa evaluation core và đo lại cùng đầu vào mà không cần sinh thêm câu trả lời mỗi lần. Điều kiện là giữ đúng QA và thứ tự chunks gắn với từng kết quả.

**Regression** trong Lab là trung bình một answer metric giảm **hơn `0.05`** so với baseline. Đây là phép so sánh giữa hai lần chạy, khác với `passed` của một QA dựa trên ngưỡng `0.5`. Bạn cần giữ riêng hai quyết định này khi viết report và đề xuất quality gate trong reflection. Repo yêu cầu chiến lược CI/CD; phần bắt buộc ở đây là hàm so sánh và báo cáo, chưa yêu cầu tạo workflow triển khai.

1. Hoàn thiện `BenchmarkRunner.run()`. Với mỗi pair, gọi `agent_fn(pair.question)`, chuyển actual answer vào `run_full_eval()` và truyền `pair.retrieved_contexts` qua tham số `contexts`. Giữ lại pair gốc trên `EvalResult`, vì adapter cần `metadata.id` để in bảng và liên kết với artifacts. Nếu tạo pair mới mà mất metadata, điểm vẫn có thể được tính nhưng bạn sẽ khó biết nó thuộc câu hỏi nào.

2. Hoàn thiện `generate_report()`, `identify_failures()` và `run_regression()`. Report cần tổng số cases, số passed, pass rate, trung bình ba answer metrics, trung bình hai retrieval metrics và thống kê failure types. Chỉ lấy trung bình retrieval trên các giá trị khác `None`; nếu toàn bộ đều thiếu, trả `None`. Đối chiếu docstring và tests cho dữ liệu rỗng, thay vì để phép chia gây lỗi.

3. Hoàn thiện `FailureAnalyzer`. `categorize_failures()` đếm theo `failure_type`; `find_root_cause()` gợi ý nguyên nhân từ scores; `generate_improvement_suggestions()` trả gợi ý cụ thể; `generate_improvement_log()` tạo bảng Markdown theo các cột đã quy định, với trạng thái `Open`. Dùng gợi ý root cause làm điểm bắt đầu kiểm tra trace. Một score thấp chưa đủ chứng minh nguyên nhân thực tế nằm ở retrieval hay generation.

4. Đồng bộ `template.py` sang `solution/solution.py`, rồi chạy nhóm tests tương ứng và toàn suite:

   ```bash
   pytest tests/test_solution.py::TestBenchmarkRunner tests/test_solution.py::TestRunRegression tests/test_solution.py::TestRetrievalMetricWiring::test_runner_forwards_retrieved_contexts tests/test_solution.py::TestRetrievalMetricWiring::test_report_includes_retrieval_averages -v
   pytest tests/test_solution.py::TestFailureAnalyzer tests/test_solution.py::TestGenerateImprovementLog -v
   pytest tests/ -v
   ```

Nhóm Runner dự kiến **11 passed**, nhóm Analyzer **9 passed**. Nếu hai retrieval averages bị thiếu, kiểm tra đường truyền từ `QAPair.retrieved_contexts` tới `run_full_eval()` trước, rồi mới kiểm tra cách tổng hợp report. Nếu regression báo sai, đối chiếu cả new và baseline cùng metric, đúng chiều giảm và đúng điều kiện "hơn `0.05`".

**Hoàn thành CP3:** toàn suite đạt **41 passed, 1 skipped** khi chưa làm bonus. Các con số này là mốc cần đạt sau khi bạn hoàn thiện code. Chạy `python template.py` chỉ thực hiện demo nhỏ có sẵn; nó không thay thế unit tests hay benchmark 20 QA. Khi required tests đã pass và hai file code đồng bộ, bạn có thể chuyển sang xây dựng dữ liệu để đánh giá hệ thống thật.

## Tạo golden dataset có evidence kiểm chứng được — CP4, phần dữ liệu

Mở `data/technology_store/manifest.json`, sau đó mở các tài liệu Markdown được liệt kê. Corpus này là dữ liệu mô phỏng của OrbitTech Store và là nguồn sự thật duy nhất cho Lab. Đầu ra của pha là `golden_dataset.json` chứa đúng 20 QA, dùng đủ 10 tài liệu và được validator chấp nhận.

**Ground truth** ở đây là expected answer do bạn viết từ evidence. **Provenance** là khả năng truy lại evidence về đúng `source_doc` và đoạn trích nguyên văn. Hai điều này bổ sung cho nhau: một đoạn trích đúng nguồn vẫn có thể không hỗ trợ toàn bộ câu trả lời bạn viết. Validator kiểm tra cấu trúc và provenance; bạn chịu trách nhiệm đọc lại ý nghĩa, điều kiện và ngoại lệ.

**Stratified sampling** trong bài là phân bổ 5 Easy, 7 Medium, 5 Hard và 3 Adversarial. Easy kiểm tra tra cứu trực tiếp; Medium yêu cầu kết hợp quy trình hoặc nhiều tài liệu; Hard xử lý điều kiện, ngoại lệ hoặc phiên bản chính sách. Độ khó cần đến từ việc suy luận trên nguồn. Kéo dài câu hỏi mà vẫn chỉ tra một thông tin không làm nó thành Hard.

1. Đọc manifest và corpus trước khi điền QA. Ghi nhận tài liệu nào hỗ trợ từng chủ đề như sản phẩm, thanh toán, vận chuyển, bảo hành, quyền riêng tư và cập nhật chính sách. Giữ corpus nguyên trạng. Khi một điều bạn định viết không có trong nguồn, điều chỉnh câu hỏi hoặc expected answer để trở về phạm vi nguồn.

2. Trong `golden_dataset.json`, giữ cấu trúc cấp cao và 20 slots có sẵn. Giữ `schema_version`, `corpus_id`, thứ tự IDs, `difficulty`, `attack_type`. Điền `question`, `expected_answer` và `contexts`. Với mỗi context, đặt `source_doc` đúng tên file và copy đoạn ngắn nguyên văn vào `text`. Có thể thêm hoặc bớt context objects theo evidence cần dùng, nhưng không để lại object rỗng. Nên viết question và answer bằng tiếng Anh để khớp corpus và prompt của trợ lý.

3. Kiểm tra ba slots Adversarial. A01 giữ `out_of_scope`; A02 giữ `prompt_injection`; A03 giữ `false_premise_or_ambiguous_trap`. Cả ba cần evidence phù hợp từ `00_system_scope.md`. Viết expected answer mô tả hành vi được chính sách hỗ trợ: giới hạn phạm vi, giữ quy tắc hệ thống hoặc xử lý tiền đề sai. Đừng dùng một câu vô nghĩa chỉ để lấp slot.

4. Tự đọc lại từng QA rồi chạy validator từ thư mục gốc. Kiểm tra mọi claim trong expected answer có evidence; các câu hỏi không trùng ý; ngày, số tiền, điều kiện và ngoại lệ quan trọng vẫn đầy đủ. Coverage toàn bộ tài liệu phải đến từ những QA có lý do sử dụng nguồn đó.

   ```bash
   python validate_golden_dataset.py
   ```

   Khi cấu trúc và provenance hợp lệ, validator in:

   ```text
   PASS: dataset structure and evidence provenance are valid.
   ```

5. Trong Exercise 3.1 của `exercises.md`, ghi phân bố, coverage và trạng thái validator. Chọn ba case đại diện, giải thích vì sao mỗi case phù hợp difficulty hoặc attack type. Dataset đầy đủ nằm trong JSON; worksheet chỉ ghi kết quả và quyết định thiết kế để người review hiểu cách bạn xây dựng nó.

Nếu validator báo `text is not a verbatim substring`, copy lại từ đúng file nguồn, giữ dấu câu và khoảng trắng. Nếu thiếu coverage, xem tài liệu còn thiếu và thiết kế case phù hợp; thêm evidence không liên quan chỉ để đủ số không giải quyết chất lượng dataset. Các lỗi `expected difficulty` hoặc `expected attack_type` cần được xử lý bằng cách khôi phục giá trị slot ban đầu.

**Hoàn thành phần dữ liệu:** validator báo PASS, phân bố đạt **5/7/5/3**, coverage **10/10** và Exercise 3.1 giải thích được ba lựa chọn thiết kế. PASS chưa xác nhận chất lượng ngữ nghĩa hay độ khó thực tế. Khi bạn đã đối chiếu thủ công expected answer với evidence, dataset mới sẵn sàng làm đầu vào sinh actual answers mà không đưa đáp án tham chiếu vào bước generation.

## Sinh actual answers và ghi benchmark — CP4, phần đánh giá

Giữ terminal ở thư mục gốc. Trước pha này, golden dataset phải PASS validator và evaluation core phải qua CP3. Bạn sẽ tạo `artifacts/actual_answers.json`, dùng nó để sinh `artifacts/benchmark_results.json`, rồi hoàn thành Exercises 3.2–3.3. Đây là lúc cần OpenAI API key đã chuẩn bị.

`domain_assistant.py` thực hiện RAG: truy xuất chunks bằng BM25, đưa chúng cùng question cho model và lưu câu trả lời kèm trace. Khi sinh answer, nó chỉ lấy `id` và `question` từ từng QA, không dùng expected answer hay gold contexts. **Data leakage** sẽ xảy ra nếu bạn đưa đáp án tham chiếu vào hệ thống đang được chấm; khi đó benchmark không còn đo khả năng trả lời từ corpus như yêu cầu.

`evaluate_answers.py` đọc lại artifacts và gọi core trong `template.py`. Bước này không gọi model để sinh câu trả lời mới và không tự chạy `LLMJudge` để tạo điểm trong Exercise 3.2. Năm metrics của bảng đến từ evaluator bạn vừa triển khai; rubric LLM-as-a-Judge là sản phẩm thiết kế riêng ở Exercise 3.3.

1. Nếu chưa có `.env`, tạo file từ `.env.example`. Trên macOS/Linux, dùng:

   ```bash
   cp .env.example .env
   ```

   Trên Windows PowerShell, dùng:

   ```powershell
   Copy-Item .env.example .env
   ```

   Mở `.env` trong editor, điền `OPENAI_API_KEY` của bạn và giữ `OPENAI_MODEL=gpt-4o-mini` theo cấu hình mẫu của repo. Nếu đã cấu hình file, kiểm tra các trường thay vì copy đè. `.env` được ignore trong Git; không đưa key vào code, worksheet, ảnh terminal hoặc commit.

2. Chạy trợ lý từ thư mục gốc và theo dõi tiến độ từng ID:

   ```bash
   python domain_assistant.py
   ```

   Mở `artifacts/actual_answers.json` sau khi hoàn tất. Danh sách `answers` phải có đủ 20 IDs khớp golden dataset; mỗi `actual_answer` có nội dung, `error` là `null`, và `retrieved_contexts` có `source_doc`, `chunk_id`, `text`, `score`. Các trường này là trace để bạn biết câu trả lời đã được tạo từ tài liệu nào. Nếu script dừng ở lỗi, sửa nguyên nhân rồi chạy lại; không điền answer bằng tay để tạo artifact có vẻ hoàn chỉnh. File từ lần chạy trước có thể vẫn còn khi lần mới lỗi; đối chiếu `generated_at` với lần chạy vừa hoàn tất trước khi dùng kết quả.

3. Khi actual answers đã đầy đủ, chạy adapter:

   ```bash
   python evaluate_answers.py
   ```

   Terminal in bảng Exercise 3.2, aggregate report và ba cases có Overall thấp nhất. Mở `artifacts/benchmark_results.json`, kiểm tra `results` có đủ 20 cases và `summary` khớp báo cáo terminal. Nếu adapter báo question khác giữa artifacts, golden dataset đã đổi sau lần sinh answers; validate rồi sinh lại answers trước khi đánh giá.

4. Trong Exercise 3.2, điền bảng và aggregate report từ lần chạy vừa kiểm tra. Giữ đủ Context Recall, Context Precision, Faithfulness, Relevance, Completeness, Overall, Passed và Failure Type. Ghi ba ID có Overall thấp nhất. Dùng các cặp metrics để đề xuất hướng điều tra: recall thấp cùng completeness thấp gợi ý thiếu evidence; recall cao nhưng precision thấp gợi ý vấn đề hạng hoặc noise. Sau đó đọc trace trước khi kết luận.

5. Trong Exercise 3.3, chọn **3–5 dimensions** theo worksheet và viết rubric **1–5** cho từng dimension. Rubric chấm yêu cầu tối thiểu hai tiêu chí; làm theo 3–5 dimensions của worksheet đáp ứng cả hai tài liệu. Mỗi mức cần mô tả hành vi quan sát được trong hỗ trợ khách hàng OrbitTech, chẳng hạn giữ đúng điều kiện chính sách hoặc xử lý thông tin riêng tư. Điền ba edge cases khó chấm và giải thích cách giảm position, verbosity, self-preference bias. Tự thiết kế tiêu chí từ corpus; không lấy độ dài câu trả lời làm bằng chứng đủ về chất lượng.

Nếu benchmark báo core chưa hoàn thiện dù tests pass, kiểm tra `template.py`: adapter import file này, còn tests ưu tiên `solution/solution.py`. Nếu retrieval scores là `None`, kiểm tra chunks trong actual artifact và đường truyền qua Runner. Bạn có thể chạy lại evaluation trên actual answers đã lưu sau khi sửa core; việc đó giữ đầu vào câu trả lời ổn định để đối chiếu thay đổi.

**Hoàn thành CP4:** hai artifacts có dữ liệu đủ 20 QA, Exercise 3.2 có năm metrics cùng ba cases thấp nhất, Exercise 3.3 có rubric và bias controls. Theo rubric chính thức, **benchmark score không quyết định điểm Lab**. Điểm được chấm dựa trên pipeline, dataset, evidence và phân tích. Giữ kết quả thực tế để dùng cho reflection, kể cả khi nhiều câu trả lời có điểm thấp.

## Viết failure analysis và chiến lược regression — CP5

Mở `reflection.md` cùng hai artifacts và `golden_dataset.json`. Bắt đầu từ ba cases thấp nhất đã ghi trong Exercise 3.2. Bạn cần giải thích bằng evidence điều gì xảy ra, đề xuất cách xử lý và nêu phép đo sẽ dùng để kiểm tra. Đầu ra của pha là báo cáo mà người review có thể truy từ kết luận về đúng câu hỏi, answer và chunks.

**Failure taxonomy** nhóm theo loại lỗi như `hallucination`, `irrelevant`, `incomplete`, `off_topic`. **Failure clustering** đi xa hơn: nhóm những cases có chung nguyên nhân có thể xử lý. Hai cases cùng metric thấp chưa chắc cùng nguyên nhân; bạn cần so sánh trace. Kỹ thuật **5 Whys** giúp nối triệu chứng với nguyên nhân, nhưng mỗi mắt xích cần evidence hoặc được ghi rõ là giả thuyết cần kiểm tra.

Gợi ý của `find_root_cause()` dựa trên scores, nên cần đối chiếu với chính sách và retrieved chunks. Trong adapter hiện tại, mỗi phần tử `results` không có trường `root_cause`. Bạn có thể gọi lại Analyzer trên các câu trả lời đã lưu để lấy output đúng case mà không phát sinh lần gọi API mới.

1. Điền phần Benchmark Results Summary: pass rate, average/min/max, phân bố failure types và nhận định dùng ít nhất hai metrics. Lấy số liệu từ artifacts của cùng lần chạy. Bảng reflection có hàng `refusal`, nhưng `run_full_eval()` không tự sinh nhãn này. Ghi số liệu của core theo thực tế; nếu thấy hành vi từ chối qua đọc answer, mô tả riêng và nêu evidence thay vì tự đổi nhãn đã đo.

2. Với từng case trong phần Top 3 Worst Failures, điền ID, question, expected answer, actual answer, scores và kiểm tra evidence. Đối chiếu gold evidence với chunks: đoạn cần thiết có được retrieve, điều kiện có bị bỏ sót, câu trả lời có thêm claim ngoài nguồn hay không. Sau khi tự viết symptom và thử chuỗi 5 Whys, lấy gợi ý của Analyzer để so sánh. Nếu cần, mở Python REPL bằng `python` từ thư mục gốc rồi dùng đoạn dưới để in gợi ý cho ba cases thấp nhất:

   ```python
   from evaluate_answers import load_evaluation_inputs
   from template import BenchmarkRunner, RAGASEvaluator, FailureAnalyzer
   pairs, answers = load_evaluation_inputs(
       "golden_dataset.json", "artifacts/actual_answers.json"
   )
   results = BenchmarkRunner().run(pairs, answers.__getitem__, RAGASEvaluator())
   analyzer = FailureAnalyzer()
   for result in sorted(results, key=lambda item: item.overall_score())[:3]:
       print(result.qa_pair.metadata["id"], analyzer.find_root_cause(result))

   ```

   Sau dòng `print`, nhấn Enter thêm một lần để gửi dòng trống và thực thi vòng lặp trong REPL. Output sẽ phụ thuộc implementation và dữ liệu của bạn. Sao chép đúng gợi ý theo ID vào reflection, rồi giải thích bạn đồng ý hay chưa đồng ý bằng trace. Nếu một case thấp điểm vẫn có `passed=True`, ghi đúng trạng thái đó và phân tích hạn chế quan sát được; không sửa số liệu để biến nó thành failure. Nếu toàn benchmark có dưới ba failures, ghi rõ số lượng thực tế và hỏi coach cách nghiệm thu yêu cầu ba failure analyses; giữ nguyên rubric và kết quả đã đo.

3. Hoàn thành Failure Clustering và Improvement Log. Trường `failure_analysis.improvement_log` trong benchmark artifact chứa bảng do hàm của bạn tạo. Đưa bảng vào reflection và đối chiếu mỗi hàng với case thực tế; nếu dùng mã `F001` theo mẫu bảng, ghi rõ QA ID tương ứng khi phân tích. Chọn ba hành động ưu tiên, nêu target metric và cách đo lại. Ưu tiên xử lý một nguyên nhân chung khi trace cho thấy nó ảnh hưởng nhiều cases.

4. Viết Regression Testing Strategy và các phần reflection còn lại. Nêu khi nào chạy `run_regression()`, bộ dữ liệu dùng so sánh, metric chặn triển khai và metric chỉ cảnh báo. Giữ contract giảm hơn `0.05` trong code; bạn có thể thảo luận mức phù hợp của ngưỡng này trong báo cáo. Đề xuất 2–3 cases cho vòng benchmark tiếp theo trong reflection, đồng thời giữ dataset nộp hiện tại đúng 20 slots để đáp ứng validator.

**Hoàn thành CP5:** ba phân tích truy được về ID và evidence, 5 Whys phân biệt quan sát với giả thuyết, improvement log có cách kiểm chứng, regression strategy nêu điều kiện chạy và xử lý kết quả. Theo `RULES.md`, bạn được dùng AI hỗ trợ giải thích và debug nhưng phải tự viết phần phân tích, hiểu code và giải thích được bài khi coach review.

Sau phần bắt buộc, bạn có thể làm Exercise 3.4 (**+5**) để so sánh hai frameworks trên cùng input, hoặc Exercise 3.5 (**+5**) để đo reranking trên ít nhất năm cases với cùng tập chunks. Ghi phương pháp và kết quả trong worksheet. Tổng bonus tối đa **+10**, chỉ xét khi phần bắt buộc hoàn thành; quy chế giới hạn tổng điểm nằm trong `RULES.md`.

## Kiểm tra và nộp repository cá nhân

Bạn nộp **link repository GitHub cá nhân lên Codelab** theo thông báo của giảng viên hoặc coach. Trước khi gửi link, kiểm tra bài trên chính repository sẽ nộp để người chấm mở được code và báo cáo.

1. Kiểm tra thư mục gốc theo mẫu `K4-L3B-HoVaTen-MSSV` và repository GitHub theo mẫu `K4-L3B-<HoVaTen>-<MSSV>-AIEvaluation`. Thay thông tin bằng họ tên, MSSV thật của bạn. Giữ các file starter cần chạy lại bài; bốn deliverables bắt buộc là:

   ```text
   K4-L3B-HoVaTen-MSSV/
   ├── solution/solution.py
   ├── golden_dataset.json
   ├── exercises.md
   └── reflection.md
   ```

2. Đồng bộ code theo CP1 và chạy kiểm tra cuối từ thư mục gốc. Mốc phần bắt buộc là **41 passed, 1 skipped** và validator **PASS**; nếu hoàn thiện bonus reranking đúng, suite đạt **42 passed**.

   ```bash
   pytest tests/ -v
   python validate_golden_dataset.py
   git status
   git diff --check
   git diff
   ```

3. Kiểm tra thay đổi rồi commit, push theo workflow Git của lớp. Không đưa `.env`, API key hoặc dữ liệu nhạy cảm lên GitHub. Hai artifacts là file nộp tùy chọn, nhưng Exercises 3.2 và reflection phải dựa trên kết quả chạy thật. Không sửa tests để đạt checkpoint.

4. Mở lại repository trên GitHub, kiểm tra bốn deliverables đã cập nhật. Để Public hoặc cấp quyền cho giảng viên/coach theo yêu cầu, rồi tự nộp link lên Codelab. Hạn mặc định là **12h trưa ngày hôm sau lab (GMT+7)**.

Rubric chính thức phân bổ điểm như sau:

| Tiêu chí | Điểm |
|---|---:|
| Core coding + required tests pass | 50 |
| Golden dataset 20 QA | 15 |
| LLM-as-a-Judge rubric design | 10 |
| Benchmark, 5 Whys, failure analysis | 15 |
| Code quality, type hints, regression strategy | 10 |
| **Total** | **100** |

Sai tên repository bị trừ **5 điểm**; commit secret bị trừ **10 điểm**. Không giải thích được phần bài làm sẽ bị hủy điểm phần đó; đạo văn khiến cả hai bên nhận **0 điểm toàn bài**.

**Hoàn thành:** link đã được nộp, người chấm truy cập được repo và kiểm tra lại được các deliverables cùng bằng chứng của bạn.
