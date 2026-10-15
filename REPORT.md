# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | Nguyen Nhat Thang |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `02727` |
| GitHub | `nhat-thang` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | 3272 / 8190 (cruise 12 m/s) |
| Số seed huấn luyện bạn dùng (`--seeds`) | 5 (seed 8190–8194) |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1.236 | 19.3 | BLOCK (R2, R3, R4, R6) |
| IDM | 1.535 | 0.0 (1/2883) | (baseline) |
| EgoMLP | 0.752–0.758 | 20.6–30.7 | BLOCK (R2–R6) |
| PercMLP | 0.615–0.621 | 0.6–2.9 | BLOCK (R2–R6) |
| PercMLP+ (sau khi sửa) | 0.816 ± 0.018 | 0.1–0.3 | RELEASE (qua R1–R7) |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

Log thô có 992 679 dòng, 9 993 quỹ đạo, bước 0,1 s. Có 383 quỹ đạo trùng hệt nhau nên bị loại, còn lại 9 610. Các bản trùng nằm ở những id cách xa nhau, nên nếu chia theo dòng hoặc theo id liền kề thì cùng một quỹ đạo có thể rơi vào cả train lẫn test và làm rò rỉ dữ liệu. Vì vậy phải chia theo quỹ đạo, sau khi đã khử trùng. Cột `Acc_LV` gần như không liên quan đến đạo hàm tốc độ xe dẫn (tương quan 0,045), trong khi cột gia tốc xe sau khớp tốt (0,929). Vì thế tôi tính gia tốc xe dẫn từ `Speed_LV` đã làm trơn rồi lấy đạo hàm: phân vị p0,1 / p1 / p50 lần lượt là −2,77 / −1,95 / 0,05 m/s². Khoảng cách bằng hiệu vị trí trừ chiều dài xe với độ lệch trung bình 4,5 m (sai số tối đa 3,6e-11), nên cột gap nhất quán.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

Chạy lại cùng mã thì 600/600 đoạn giống nhau từng bit. Bản refactor float32 khác bit ở cả 600/600 đoạn, nhưng vượt dung sai (atol = rtol = 1e-3) ở 0 đoạn, với sai khác lớn nhất 9,3e-5 m/s². Đây là nhiễu do làm tròn, giống ví dụ (0,1+0,2)+0,3 ≠ 0,1+(0,2+0,3). Thay đổi AEB làm lệch đúng 6 đoạn (174, 179, 195, 231, 298, 517), với sai khác lớn nhất 1,98 m/s². Chỉ những khung có khoảng cách/TTC đủ nhỏ để kích hoạt AEB mới đổi lệnh; các khung bình thường vẫn giống hệt nhau. Ở chế độ phát nhanh nhất có 0 khung trễ và 0 khung rớt. Ở chế độ thời gian thật có 585 khung trễ, 581 khung rớt, và 377/600 đoạn vượt dung sai (lệch tới 7,9 m/s²). Chế độ phát nhanh che đi vấn đề thời gian của hệ thống thật.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

Tốc độ cruise 12 m/s; lưới gồm 60 khoảng cách × 45 tốc độ tiến gần = 2 700 ô. Phanh tối đa (9 m/s²) vẫn không tránh được ở 2,2 % số ô khi trễ phản ứng 0 s và 7,4 % khi trễ 0,6 s. Mô phỏng khớp với biên vật lý `unavoidable` của tôi: 0 ô lệch ở xa biên. Vùng đâm của IDM tăng từ 2,2 % lên 7,4 % khi trễ tăng từ 0 lên 0,6 s, tức là đúng bằng vùng không thể tránh. Log không phủ vùng khó: tôi khai thác được 614 sự kiện xe dẫn phanh trên 9 610 quỹ đạo (6,4 %), với đỉnh phanh p50 là −2,5 và p99 là −4,2 m/s². Người lái mạnh nhất trong log chỉ phanh 3,6 m/s² (phân vị 99,9), trong khi 9 % lưới đòi hỏi phanh mạnh hơn mức đó.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

| bộ điều khiển | L2 vòng hở | hạng | va chạm vòng kín | hạng |
|---|---|---|---|---|
| const-vel | 1,236 m | 3 | 19,3 % | 3 |
| IDM | 1,535 m | 4 | 0,0 % | 1 |
| EgoMLP | 0,755 m | 2 | 24,3 % | 4 |
| PercMLP | 0,619 m | 1 | 1,2 % | 2 |

Các cặp đổi thứ tự giữa hai cách chấm: (const-vel, IDM), (const-vel, EgoMLP), (IDM, EgoMLP), (IDM, PercMLP). IDM có L2 tệ nhất vì nó không bắt chước người lái: nó giữ khoảng cách an toàn theo luật riêng nên quỹ đạo lệch khỏi log, nhưng nó luôn phản ứng với khoảng cách thật nên chỉ đâm 1/2 883 lần. EgoMLP chỉ thấy trạng thái của xe mình, không thấy xe dẫn. Nó đạt L2 thấp nhờ ngoại suy quán tính trên 95 % cửa sổ bình thường, nhưng không thể biết xe dẫn đang phanh. Vì vậy trên lát cắt phanh nó đâm 57,8 %, và trên toàn bộ log đâm 20,6–30,7 %, tệ hơn cả constant-velocity (19,3 %).

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

Qua 5 seed, tỷ lệ va chạm của EgoMLP dao động từ 20,6 % (seed 8190, Wilson CI95 [19,2; 22,1]) đến 30,7 % (seed 8193, CI95 [29,1; 32,4]); L2 thì gần như không đổi (0,752–0,758 m). Khoảng Wilson của từng seed hẹp (khoảng ±1,6 điểm với n = 2 883) và hai khoảng này không chồng lên nhau. Điều đó cho thấy chênh lệch đến từ chính seed huấn luyện, không phải từ nhiễu do đếm ít mẫu. PercMLP cũng vậy: bốn seed ở mức 0,6–0,8 % nhưng seed 8193 lên 2,9 %. Nếu chỉ chạy một seed, tôi có thể báo "PercMLP đâm 0,6 %, gần đạt IDM" hoặc "2,9 %" tùy may rủi. CI của một seed chỉ đo nhiễu mẫu, không đo được nhiễu huấn luyện, nên kết luận sẽ tự tin quá mức. Vì thế cổng dùng seed tệ nhất (R2) và dải qua các seed (R5).

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

Người lái trong log giữ thời khoảng cách trung vị 1,85 s. Tôi hiệu chỉnh xe sau IDM được T = 1,4 s, với khoảng cách W1 giữa hai phân bố headway là 0,36, so với 0,90 của một xe không phản ứng. Như vậy xe sau đã khá chân thực, dù vẫn chưa khớp hoàn toàn.

| t_react = 0,6 s | xe sau phát lại | xe sau phản ứng | (va chạm phía sau: phát lại / phản ứng) |
|---|---|---|---|
| const-vel | 77,6 % | 77,6 % | 0,0 % / 0,0 % |
| IDM | 75,7 % | 7,4 % | 68,3 % / 0,0 % |
| EgoMLP | 82,0 % | 82,0 % | 0,0 % / 0,0 % |
| PercMLP | 75,7 % | 28,4 % | 48,3 % / 0,0 % |

(Với t_react = 0: IDM có 75,3 % → 2,2 %, trong đó 73,0 % là va chạm phía sau.) IDM phanh đúng khi bị cắt vào, nhưng xe sau phát lại theo log không biết điều đó nên lao vào đuôi xe thử, và lỗi bị tính cho IDM. Chỉ những bộ không phanh (const-vel, EgoMLP) mới không bị đâm từ phía sau. Trong thế giới phát lại, IDM trông tệ ngang const-vel (75,7 % so với 77,6 %), dẫn tới kết luận sai. Khi xe sau biết phản ứng, IDM chỉ còn đúng 7,4 % vùng không thể tránh.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

PercMLP qua R1 và R7 nhưng rớt R2 (seed tệ nhất 2,9 %, cận trên 3,6 % so với 0,0 % + 1 điểm), R3 (10,6 % trên lát cắt phanh), R4 (21,0 % lưới cắt vào so với ngưỡng 2 %), R5 (dải 0,6–2,9 %) và R6 (28,4 % so với baseline 7,4 %). Nguyên nhân gốc là phân bố huấn luyện không chứa vùng khó: log gần như không có cắt vào gấp, và 9 % lưới đòi hỏi phanh mạnh hơn 3,6 m/s² mà người lái trong log chưa bao giờ làm. Ngoài ra, khi tự lái, mô hình đi vào những trạng thái nó chưa từng thấy (covariate shift). Tôi sửa bằng hai cách. Thứ nhất, thêm 196 890 cửa sổ từ 20 000 tình huống cắt vào mô phỏng, gắn nhãn bằng IDM. Thứ hai, chạy 2 vòng DAgger: gắn nhãn IDM ở chính các trạng thái mô hình tự đi vào, tổng cộng khoảng 894 000 cửa sổ mỗi seed. PercMLP+ qua cả 7 quy tắc: va chạm trên log 0,1–0,3 %, lưới 0,7 %, thế giới phản ứng 8,2 %, và ở tốc độ cruise chưa thấy khi huấn luyện (20 m/s) còn 1,1–2,3 % thay vì 21,3–35,9 %. L2 vòng hở lại tăng từ 0,619 lên 0,816 m, tức là điểm vòng hở xấu đi đúng lúc mô hình an toàn hơn. Dù vậy, PercMLP+ **không tốt hơn** IDM: nó học từ IDM nên khó vượt thầy, và vẫn tệ hơn IDM ở mọi chỉ số (0,3 % so với 0,0 %, 8,2 % so với 7,4 %, 1,1–2,3 % so với 0,0 % ở 20 m/s). Tôi không đề xuất thay IDM ngay. Trước khi quyết định nên chạy shadow mode bên cạnh IDM, thử thêm các kịch bản ngoài phân phối và dữ liệu thật, và chỉ thay khi có lợi ích rõ ràng mà IDM không đem lại.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

1. **Thế giới chỉ có một chiều dọc, không có cảm biến thật.** Mô hình nhận khoảng cách và tốc độ gần như hoàn hảo (nhiễu thử chỉ ±1 m, ±0,5 m/s, khiến PercMLP lên 0,7–3,0 %). Hệ thật có lỗi phát hiện, mất dấu, đổi làn và va chạm ngang, tức là những lỗi không xuất hiện trong 2 700 ô lưới của tôi.
2. **Xe dẫn trong log không phản ứng, và tác nhân phản ứng là giả định của chúng ta.** Xe sau IDM với T = 1,4 s vẫn lệch khỏi người lái (W1 = 0,36), và kết quả của IDM đổi từ 75,7 % sang 7,4 % chỉ vì giả định này. Kết luận phụ thuộc mạnh vào một mô hình người lái chưa được kiểm chứng.
3. **Vùng khó vẫn chủ yếu là mô phỏng.** Chỉ có 614 sự kiện phanh thật, đỉnh p99 là −4,2 m/s². PercMLP+ "qua cổng" nhờ dữ liệu mô phỏng gắn nhãn bằng chính IDM, nên cổng chủ yếu đo xem nó bắt chước IDM tốt đến đâu trong chính mô phỏng của chúng ta, không đo độ an toàn thật.
4. **Thống kê còn mỏng.** Lát cắt phanh chỉ có n = 199, nên CI rộng (PercMLP+ 1,0 %, cận trên 3,6 %). Năm seed chưa đủ để ước lượng đuôi phân phối huấn luyện, và 2 883 quỹ đạo không đủ để chứng minh tỷ lệ sự cố hiếm ở mức cần cho phát hành.

Việc cần làm tiếp: thêm cảm biến và nhiễu thực tế, mô phỏng 2D có đổi làn và nhiều tác nhân, hiệu chỉnh tác nhân phản ứng trên dữ liệu tương tác thật, thu thêm log có sự kiện nguy hiểm, chạy shadow mode trên đường, và tăng số seed/kịch bản để thu hẹp khoảng tin cậy.
