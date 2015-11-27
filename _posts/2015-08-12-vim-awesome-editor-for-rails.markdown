---
layout: post
title:  "VIM - awesome editor for rails"
date:   2015-08-12 23:55:00
summary: VIM là 1 công cụ soạn thảo tuyệt vời cho rails. VIM cung cấp khả năng tùy biến cao theo thói quen của người dùng giúp tăng tốc độ soạn thảo cho lập trình viên.
description: VIM là 1 công cụ soạn thảo tuyệt vời cho rails. VIM cung cấp khả năng tùy biến cao theo thói quen của người dùng giúp tăng tốc độ soạn thảo cho lập trình viên.
categories: [ruby on rails]
tags: [""]
images: /images/chimen.png
author: Lân Nguyễn
---

**VIM** là 1 công cụ soạn thảo tuyệt vời cho rails. VIM cung cấp khả năng tùy biến cao theo thói quen của người dùng giúp tăng tốc độ soạn thảo cho lập trình viên. Vì vậy trong bài viết này tôi sẽ hướng dẫn bạn cách biến VIM như 1 IDE thông qua 3 bước đơn giản sau.

Đây là kết quả sau khi cấu hình VIM thành 1 IDE:

![VIM IDE](/images/chimen.png)

Màn hình IDE gồm 4 window: editor, server, deploy, ssh

- Editor được chia làm 2 pane:

  + Pane phía trên là VIM editor với cây thư mục (NERTree) và khu vực soạn thảo chính
  + Pane phía dưới là 1 terminal để gõ lệnh git, chạy test hoặc console
- Server: Mặc định khi mở IDE sẽ chạy rails s
- Deploy: Dùng để deploy capistrano
- SSH: ssh lên remote server như digital ocean


__1. VIM basic__

VIM có 4 chế độ: normal mode, insert mode, visual mode, command mode:

- Normal mode: Dùng chế độ này để di chuyển tới qua các vị trí và thay đổi text bằng lệnh.  Dùng ESC để quay về chế độ normal
- Insert mode: Dùng chế độ này để chèn thêm text. Dùng i để vào chế độ insert
- Visual mode: Dùng chế độ này để chọn 1 vùng text và thay đổi text trên vùng đã chọn bằng lệnh. Dùng v hoặc V để vào chế độ visual
- Command mode: Dùng chế độ này để thực hiện lệnh trong VIM và ngoài hệ thống. Dùng : để vào chế độ command

Thao tác trên VIM hoàn toàn sử dụng **bàn phím** thay vì chuột và bàn phím như các editor khác. Điều này giúp người sử dụng VIM tập trung tối đa bàn tay vào các phím. Trong khi người sử dụng các editor thông thường sẽ phải sử dụng tay ở chuột để di chuyển, dùng bàn phím để sửa, có lúc lại sử dụng bàn phím để di chuyển. Điều này thực sự gây cản trở tốc độ  và sự tập trung của người viết. Dưới đây là danh sách chi tiết các nút trên bàn phím được sử dụng trong VIM

![VIM Cheatsheet](http://www.viemu.com/vi-vim-cheat-sheet.gif)

  Phần quan trọng nhất khi 1 lập trình viên viết chương trình có lẽ là việc **di chuyển(motion)** giữa các dòng code, giữa các file để đọc hiểu hệ thống và tìm ra vị trí thích hợp nhất để thêm, sửa hoặc xoá. Vì vậy, VIM đã tập trung phần lớn các phím cho việc di chuyển này. Các phím di chuyển được đánh dấu bằng **màu xanh lá cây**.

  Việc di chuyển được thực hiện theo nhiều cấp độ khác nhau:

  - Di chuyển trái 1 kí tự (h), phải 1 kí tự (l), xuống (j), lên (k)
  - Để di chuyển nhanh hơn, chúng ta dùng w - di chuyển theo từ, W - di chuyển theo từ lớn, b - di chuyển theo từ nhưng chiều quay lại, B - di chuyển theo từ lớn nhưng chiều quay lại
  - Để tìm kiếm 1 kí tự trên 1 dòng: f + kí tự muốn tìm
  - Để tìm kiếm 1 từ trên toàn file: / + từ muốn tìm
  - Để di chuyển quay lại các vùng mà con trỏ đã đi qua sử dụng Ctrl + o, để nhảy tới dùng Ctrl + i
  - Để tìm kiếm từ trong toàn project hoặc 1 thư mục: Sử dụng plugin [https://github.com/rking/ag.vim](https://github.com/rking/ag.vim) (phần 2)

  Một phần khác biệt nữa giữa VIM và các editor khác là sử dụng lệnh để thay đổi text. Với mỗi lệnh thì trên VIM sẽ đều có 2 phiên bản chữ thường và chữ hoa. Chữ hoa thường mang ý nghĩa phạm vi rộng hơn hoặc ngược lại với chữ thường. Ngoài ra, các lệnh thường đi kèm với 1 motion để có thể thực hiện trong 1 phạm vi rộng hơn. Công thức: **Command + Motion**. Dưới đây chúng ta sẽ đi vào các lệnh cơ bản theo cặp

 - Chèn text:

   + Cặp i/I: i - bắt đầu chèn text tại vị trí con trỏ, I - vào chế độ insert ở đầu dòng
   + Cặp a/A: a - bắt đầu chèn text sau vị trí con trỏ, A - bắt đầu chèn text ở vị trí cuối dòng
   + Cặp o/O: o - bắt đầu chèn text ở dòng phía dưới, O - bắt đầu chèn text ở dòng phía trên
   + Cặp c/C: c + motion (w, W…) - xoá số kí tự chọn theo motion và vào chế độ insert, C - xoá kí tự cả 1 dòng và vào chế độ insert
   + Cặp s/S: s - thay thế 1 kí tự, S - thay thế cả dòng
   + Cặp x/X: x - xoá kí tự ở vị trí con trỏ, X - xoá kí tự phía trước con trỏ
   + Cặp d/D: d + motion (w, W…) - xoá số kí tự theo motion, D - xoá cả dòng


 - Copy và paste:

   + Cặp y/Y: y + motion - copy số kí tự theo motion vào buffer, Y - copy cả dòng vào buffer
   + Cặp p/P: p - paste từ buffer vào sau con trỏ, P - paste từ buffer vào trước vị trí con trỏ

  Một cặp khác cũng được sử dụng rất phổ biến ở các trình soạn thảo là undo/redo. Trong VIM ta sử dụng u cho undo và Ctrl + r cho redo

  Điều cuối cùng, ma thuật trong VIM nằm ở trong việc lặp lại sự thay đổi. Việc này giúp cho việc thay đổi text xuất hiện nhiều lần hoặc 1 vùng text có 1 quy luật nhất định với 1 tốc độ nhanh khủng khiếp với số thao tác ít đến mức không thể tin được

  - Với việc lặp đơn giản, chúng ta sử dụng phím . để lặp lại lệnh đã thực hiện trước
  - Với 1 tập lệnh phức tạp hơn để chỉnh sử 1 vùng, ta sẽ lưu các biến đó vào 1 biến macro để sau đó lấy tập lệnh từ biến đã lưu và áp dụng vào 1 vùng mới tương tự vùng cũ. Để bắt đầu record lại các bước, ta sử dụng q + a (biến sẽ lưu các bước). Để áp dụng (play) các bước trên ta chọn vùng rồi nhấn @ + a. Link tham khảo demo: [http://www.thegeekstuff.com/2009/01/vi-and-vim-macro-tutorial-how-to-record-and-play/](http://www.thegeekstuff.com/2009/01/vi-and-vim-macro-tutorial-how-to-record-and-play/)

__2. VIM plugins__

  Với VIM thuần, ta có được các việc thay đổi cơ bản dựa trên bàn phím. Nhưng muốn có tốc độ nhanh hơn nữa, tuỳ theo ngôn ngữ hay framework lập trình mà sẽ có những thư viện phục vụ riêng. Mục đích của việc cài các plugin này cũng theo tinh thần VIM tập trung tối đa vào bàn phím trên 1 cửa sổ. Để cài được các plugin cho vim ta dùng VIM [Vundle](https://github.com/VundleVim/Vundle.vim), rồi khai báo các plugin như kiểu Gemfile trong rails. Đối với ruby và rails, ta có 1 số plugin sau:

  - [NERDTree](https://github.com/scrooloose/nerdtree): Cây thư mục cho VIM.
  - [Ctrlp](https://github.com/kien/ctrlp.vim): Tìm kiếm tên file theo phong cách Ctrl + p.
  - [Vim-rails](https://github.com/tpope/vim-rails): VIM theo phong cách rails.
  - [Vim-fugitive](https://github.com/tpope/vim-fugitive): Tích hợp các lệnh của Git trong VIM
  - [Ag.vim](https://github.com/rking/ag.vim): Giúp tìm kiếm từ trong thư mục hoặc cả project

  Một plugin tuy rất nhỏ nhưng rất hữu ích khi dùng đó là **vim-trailing-whitespace**. Plugin giúp xoá các ký tự space ở cuối hoặc đầu dòng trong file. Việc này giúp người đọc code không bị nhiễu loạn bởi những kí tự không cần thiết.

  Ngoài ra, còn rất nhiều plugin khác, các bạn có thể tham khảo ở những nguồn sau:

- [https://github.com/thoughtbot/dotfiles/blob/master/vimrc.bundles](https://github.com/thoughtbot/dotfiles/blob/master/vimrc.bundles)
- [http://vimawesome.com/](http://vimawesome.com/)
- [https://github.com/lannn/dotfiles/blob/master/vimrc.local](https://github.com/lannn/dotfiles/blob/master/vimrc.local)


__3. VIM với Tmux__

  Tmux giúp quản lý các session độc lập. Tmux giúp tạo các window, panes với mục đích bổ trợ trong quá trình phát triển ứng dụng rails như chạy rails c, rails s, ssh, deploy…
  Có 1 gem giúp config các window và panes theo 1 định nghĩa trước gọi là tmuxinator [(https://github.com/tmuxinator/tmuxinator)](https://github.com/tmuxinator/tmuxinator). Dưới đây là ví dụ config 1 project:

{% highlight yaml %}

# ~/.tmuxinator/chimen.yml

name: chimen
root: ~/code/chimen

# Optional tmux socket
# socket_name: foo

# Runs before everything. Use it to start daemons etc.
# pre: sudo /etc/rc.d/mysqld start

# Runs in each window and pane before window/pane specific commands. Useful for setting up interpreter versions.
pre_window: rvm use 2.1.2@chimen

# Pass command line options to tmux. Useful for specifying a different tmux.conf.
# tmux_options: -f ~/.tmux.mac.conf

# Change the command to call tmux.  This can be used by derivatives/wrappers like byobu.
# tmux_command: byobu

windows:
  - editor:
      layout: b256,204x42,0,0[204x30,0,0,0,204x11,0,31,4]
      panes:
        - vim
        - clear
  - server: bundle exec rails s -p 5000
  - deploy:
  - ssh:
{% endhighlight %}

  Với config như trên thì khi gọi: **mux chimen** sẽ tạo ra 1 tmux tên là chimen với 4 cửa sổ

  - Window đầu tiên là editor được chia làm 2 pane, pane trên là vim editor chính, pane dưới là phần chạy terminal bổ trợ
  - Window thứ 2 chạy server
  - Window thứ 3 chạy ssh
  - Window thứ 4 chạy deploy

Trước khi chạy mỗi window, mỗi terminal đều trigger ruby version 2.1.2 và gemset chimen

  Cuối cùng, để tăng tốc với VIM thì cũng giống như bạn dùng bất kỳ chương trình nào là phím tắt. Dưới đây là một số cách map phím tắt mà mình thường dùng:

  - Đầu tiên, vì phím "ESC" chuyển từ các chế độ khác về normal mode trong VIM được sử dụng rất thường xuyên, mà vị trí của phím đó khá xa ngón tay của bạn trên bàn phím. Để phím n gần ngón tay của người hơn, ta map phím **"ESC" thành phím "CAPSLOCK"**. Hướng dẫn trên Linux - [http://askubuntu.com/questions/363346/how-to-permanently-switch-caps-lock-and-esc](http://askubuntu.com/questions/363346/how-to-permanently-switch-caps-lock-and-esc), trên Mac - [http://stackoverflow.com/questions/127591/using-caps-lock-as-esc-in-mac-os-x](http://stackoverflow.com/questions/127591/using-caps-lock-as-esc-in-mac-os-x)
  - Để gia tăng được các phím tắt ta map "Leader" to "Space" trong ~/.vimrc: **let mapleader = " "**
  - Đối với những file hay di chuyển đến trong mọi project rails như application_controller.rb, application_helper.rb, application.js, application.css... Ta nên map thành theo các phím tắt. Dưới đây là danh sách map của mình:

{% highlight yaml %}
map <Leader>q :q<CR>
map <Leader>w :w<CR>
map <Leader>sv :source ~/.vimrc<CR>
map <Leader>bi :PluginInstall<CR>
map <Leader>vi :tabe ~/.vimrc.local<CR>
map <Leader>ct :!ctags -R<CR>
map <Leader>sg :sp<CR>:grep<SPACE>
map <Leader>te :tabe<SPACE>
map <Leader>f :FixWhitespace<CR>
map <Leader>nt :NERDTree<CR>
map <Leader>ag :Ag<SPACE>
map <C-c> "+y<CR>
map <Leader>v "+p<CR>
nnoremap <S-CR> O<Esc>j
nmap <CR> o<Esc>k
map <Leader>rm :Remove!<CR>
map <Leader>n <C-w>v<C-h><SPACE><SPACE>
map <Leader>fu :%s/<C-v><C-m>//<CR>
map <C-a> <ESC>gg<S-v>G<CR>

" Git
map <Leader>gs :Gstatus<CR>
map <Leader>gd :Gvdiff<CR>
map <Leader>gac :Gcommit -am ""<LEFT>
map <Leader>gc :Gcommit -m ""<LEFT>
map <Leader>co :Gread<CR>
map <Leader>ga :Gwrite<CR>
map <Leader>gb :Gblame<CR>
map <Leader>gm :Gmove<SPACE>
map <Leader>gr :Gremove<CR>
map <Leader>gl :Glog<CR>

" Rails
map <Leader>bb :!bundle install<CR>
map <Leader>sc :sp db/schema.rb<CR>
map <Leader>m :Rmodel<SPACE>
map <Leader>c :Rcontroller<SPACE>
map <Leader>vc :RVcontroller<SPACE>
map <Leader>vm :RVmodel<SPACE>
map <Leader>vv :RVview<SPACE>
map <Leader>av :AV<CR>
map <Leader>rv :RV<CR>
map <Leader>ac :Rcontroller application<CR>
map <Leader>rt :e config/routes.rb<CR>
map <Leader>rp :e config/routes_premium.rb<CR>
map <Leader>ah :RVhelper application<CR>
map <Leader>g :e Gemfile<CR>
map <Leader>a <ESC>ggVG<CR>
map <Leader>db :e config/database.yml<CR>
map <Leader>a <ESC>ggVG<CR>
map <Leader>aj <ESC>:e app/assets/javascripts/application.js<CR>
map <Leader>as <ESC>:e app/assets/stylesheets/application.scss<CR>
map <Leader>d <ESC>obinding.pry<ESC>
map <Leader>dv <ESC>o<% binding.pry %><ESC>
map <Leader>dh <ESC>o- binding.pry<ESC>
map <Leader>u <ESC>vawgU
map <Leader>gl <ESC>vawgu
map <Leader>b :e Bowerfile<CR>
{% endhighlight %}

Trên đây là cách cài đặt và cấu hình để biến VIM thành 1 IDE toàn diện cho dự án rails. Hi vọng các bạn sẽ cảm thấy hạnh phúc và làm việc hiệu quả hơn với VIM trong những dự án rails sắp tới.

Nguồn tham khảo:

1. Cuốn [Practical VIM](http://www.cs.csubak.edu/~jhernandez/files/PracticalVim.pdf)
2. [http://vimcasts.org/](http://vimcasts.org/)
3. [https://github.com/thoughtbot/dotfiles](https://github.com/thoughtbot/dotfiles)


**Lân Nguyễn**
