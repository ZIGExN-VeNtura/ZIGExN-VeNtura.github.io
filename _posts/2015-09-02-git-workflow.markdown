---
layout: post
title:  GIT best practice
date:   2015-09-02 23:55:00
summary: Ngày nay, trong quá trình làm việc nhóm trong 1 dự án code thì GIT là 1 công cụ không thể thiếu được. GIT giúp từng cá nhân theo dõi, phát triển từng chức năng riêng biệt. Đồng thời, GIT cũng hỗ trợ tốt trong việc làm việc nhóm, giúp hợp nhất các đóng góp của mỗi thành viên trong nhóm
categories: [web development]
tags: ["git"]
images: /images/socialite.jpg
author: Lân Nguyễn
---

![GIT COVER](/images/socialite.jpg)

Ngày nay, trong quá trình làm việc nhóm trong 1 dự án code thì **GIT** là 1 công cụ được sử dụng hàng ngày. GIT giúp từng cá nhân theo dõi, phát triển từng chức năng riêng biệt. Đồng thời, GIT cũng hỗ trợ tốt trong việc làm việc nhóm, giúp hợp nhất các đóng góp của mỗi thành viên trong nhóm. Nhưng để sử dụng tốt công cụ tuyệt vời này, tôi xin đưa ra 1 số best practice sau.


__1. Khi 1 thành viên phát triển độc lập 1 chức năng__

- Tiêu chuẩn:

  + Mỗi commit nên tập trung làm 1 việc (số lượng file vừa phải)

  + Nội dung message rõ ràng không chung chung như "Fix review", "Fix bug"...

Thông thường sẽ có 2 thời điểm chúng ta phát hiện cần sửa 1 commit:

  + Thứ nhất, **khi vừa commit xong**, ta phát hiện ra còn thiếu file hoặc cần sửa gấp 1 vài đoạn nhỏ trong code (như chưa xóa debugger, chưa đặt method vào vùng private...)
Cách sửa gấp:
 
  Sửa các file đó
  Add vào repo: git add .
  Thêm file đó vào commit trước đó: git commit --amend. Lệnh này sẽ đưa ra màn hình editor với nội dung gồm commit message của commit trước và các file của commit cũ cộng với các file mới thêm vào. Từ đây ta có thể sửa commit message cũ hoặc không, sau đó sẽ thoát ra. Git sẽ tạo ra 1 commit với id mới bao gồm tất cả các file cũ và mới

  + Thứ hai, sau 1 thời gian feature branch có rất nhiều commit, chúng ta sử dụng git rebase -i (Interactive Rebase) để giải quyết các vấn đề sau:
![GIT REBASE](/images/rebase.png)

![GIT REBASE](/images/default_rebase.png)

 Một số commit có message chưa rõ nghĩa, cần sửa lại message
![VIM REBASE REWORD](/images/reword_rebase.png)

 Một số commit quá lớn cần tách ra làm nhiều commit nhỏ hơn
![VIM REBASE SPLIT](/images/split_rebase.png)

![VIM REBASE SPLIT](/images/split_rebase_1.png)

![VIM REBASE SPLIT](/images/split_rebase_2.png)

![VIM REBASE SPLIT](/images/split_result.png)

 Một số commit quá nhỏ, có thể gom lại với nhau
![VIM REBASE SQUASH](/images/squash_rebase.png)

![VIM REBASE SQUASH](/images/squash_rebase_1.png)

![VIM REBASE SQUASH](/images/squash_rebase_2.png)

![VIM REBASE SQUASH](/images/squash_rebase_3.png)

![VIM REBASE SQUASH](/images/squash_result.png)

 Một số commit không theo đúng thứ tự, cần sắp xếp lại
![VIM REBASE REORDER](/images/reorder_rebase.png)

![VIM REBASE REORDER](/images/reorder_result.png)
   
  
__2. Khi chức năng cần cập nhật từ phần chung khi xảy ra conflict hoặc lấy phần mới từ người khác__

- Sau khi các thành viên trong team phát triển độc lập chức năng của mình, tạo pullrequest để merge vào branch chính. 
Khi đó sẽ xảy ra việc hai hay nhiều thành viên cùng sửa chung 1 dòng code dẫn đến việc xảy ra conflict. Vì vậy để ngăn trước trường hợp này xảy ra thì việc chia vùng chức năng với mức giảm ràng buộc giữa các chức năng với nhau sẽ giúp cho giảm thiểu conflict code giữa các thành viên trong team.

Có 2 cách để lấy code từ branch chính về feature branch là dùng: git rebase hoặc git merge

Có rất nhiều quan điểm cũng như tranh cãi giữa việc sử dụng 2 cách rebase và merge này. Nhưng chúng ta cùng đi phân tích điểm lợi, điểm hại của 2 phương pháp này:

+ Git merge: 
  Merge code giữa 2 branch và sắp xếp các commit message tính từ điểm mà feature branch đó checkout từ branch chính, đồng thời tạo 1 merge commit
  Để đồng bộ với remote server (github, bitbucket): git push origin feature_branch
+ Git rebase: 
  Move các commit tính từ thời điểm checkout ra 1 branch tạm thời
  Chuyển commit branch chính sang feature branch
  Chuyển các commit từ branch tạm về đỉnh của feature branch

Vậy git rebase sẽ re-commit các commit cũ, sắp xếp các commit theo thứ tự merge vào branch chính, đồng thời sẽ không tạo ra commit mới (vì merge commit thì rất là tệ, gây rối loạn pullrequest)
  Để đồng bộ với remote: git push origin feature_branch -f. Option -f mang nghĩa là ép buộc server phải sync với local vì các commit cũ đã thay đổi sang id mới

Một câu hỏi đặt ra: Khi nào dùng rebase và merge?

Khi feature branch được "public" cho các thành viên khác để chạy test chẳng hạn, đồng thời team là remote thì chúng ta nên sử dụng merge để người khác không bị nhầm lẫn và tiếp tục tốt công việc

Khi feature branch vẫn còn ở trạng thái "private", tức chỉ thành viên đó chỉnh sửa và team in place thì chúng ta sử dụng rebase để pullrequest giảm thiểu được số commit merge không cần thiết

**Lân Nguyễn**
