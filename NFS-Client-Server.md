##Thiết lập NFS Mount trên Ubuntu 14.04


###Giới thiệu

NFS hay Network File System, là giao thức hệ thống tập tin phân tán cho phép bạn có thể mount các thư mục từ xa trên server của bạn. Điều này cho phép bạn tận dụng không gian lưu trữ ở một vị trí khác và để viết một cách dễ dàng đến cùng một không gian từ nhiều máy chủ. NFS hoạt động tốt cho các thư mục và sẽ phải được thư mục thường xuyên.

Trong hướng dẫn này, chúng tôi sẽ giới thiệu cách cấu hình NFS mount trên máy chủ Ubuntu 14.04

###Điều kiện kiên quyết.

Trong hướng dẫn này, chúng tôi sẽ cấu hình chia sẻ các thư mục giũa hai máy chủ Ubuntu 14.04. Chúng có thể có kích thước bất kỳ. Đối với mỗi máy chủ này, bạn sẽ phải có một tài khoản được thiết lập với quyền ưu tiên sudo. Bạn có thể tìm hiểu cách cấu hình một tài khoản như vậy bằng cách làm theo 1 – 4 trong hướng dẫn thiết lập ban đầu cho các máy chủ Ubuntu 14.04.

Mục đích của hướng dẫn này là chúng ta sẽ đề cập tới máy chủ sẽ chia sẻ các thư mục của nó với máy chủ khác và máy chủ (host) sẽ gắn các thư mục này với máy khách (client).

Để giữ các thông tin này trong suốt hướng dẫn, tôi sẽ sử dụng cá địa chỉ IP sau đây như các giá trị cài đặt máy chủ:

Host: 192.168.56.21

Client 192.168.56.22

###Tải về và cài đặt các thành phần.

Trước khi chúng ta bắt đầu, bạn cần cài đặt các gói cần thiết trên cả 2 máy server host và server client.

Trên máy chủ host, bạn cần cài đặt các gói nfs-kernel-server, sẽ cho phép họ chia sẻ những thư mục của họ. Đây là thao tác đầu tiên mà chúng tôi thực hiện với apt trong phiên này, chúng tôi sẽ làm mới các gói trước khi cài đặt:

root@server:~# apt-get update

root@server:~# apt-get install nfs-kernel-server

Một khi những gói này được cài đặt, bạn có thể chuyển sang máy client.

Trên máy khách, chúng ta sẽ phải cài đặt một gói gọi là nfs-common, cung cấp chức năng NFS mà không cần phải bao gồm các thành phần của máy host. Một lần nữa, chúng tôi sẽ làm mới các gói cài đặt:

root@client:~# apt-get update
root@client:~# apt-get intstall nfs-common

###Tạo thư mục chia sẻ trên Máy chủ Host

Chúng ta sẽ thử nghiệm việc chia sẻ hai thư mục riêng trong hướng dẫn này. Thư mục đầu tiên chúng ta chia sẻ sẽ là thư mục /home có chứa dữ liệu người dùng. Thứ hai là thư mục đích chung mà chúng ta sẽ tạo ra đặc biệt cho NFS để chúng ta chứng minh cách thủ tục và các thiết lập phù hợp. Điều này sẽ được đặt tại /var/nfs.

Kể từ thư mục /home đã tồn tại, bắt đầu bằng cách tạo thư mục /var/nfs:

root@server:~# mkdir /var/nfs

Bây giờ, chúng ta có một thư mục được thiết kế đặc biệt cho mục đích chia sẻ với các máy chủ từ xa. Tuy nhiên, quyền sở hữu thư mục không phải lý tưởng. Chúng ta nên cung cấp cho người dùng quyền sở hữu cho người dùng trên hệ thống của chúng tôi được đặt tên là nobody. Chúng ta nên cung cấp quyền sở hữu nhóm cho một nhóm trong hệ thống của chúng tôi có tên là nogroup.

Chúng ta có thể làm điều đó bằng cách gõ lệnh này:

root@server:~# chown nobody:nogroup /var/nfs

Chúng tôi chỉ cần thay đổi quyền sở hữu trên các thư mục của chúng tôi được sử dụng đặc biệt để chia sẻ. Ví dụ, chúng tôi không muốn thay đổi quyền sở hữu thư mục /home của chúng tôi vì nó sẽ gây ra các sự cố cho bất kỳ người dùng nào chúng tôi có trên máy chủ.

###Cấu hình NFS Export trên Host Server

Bây giờ chúng ta có thư mục của chúng ta được tạo và gán, chúng ta có thể lướt vào tập tin cấu hình NFS để thiết lập chia sẻ các tài nguyên này:

Mở tập tin /etc/exports trong trình soạn thảo của bạn với quyền root:

root@server:~# nano /etc/exports

Các tập tin mà bạn thấy sẽ có một số comment cho thấy cấu trúc chung của mỗi dòng cấu hình. Về cơ bản, cú pháp giống như:

directory_to_share		client(share_option1,....,share_optionN)

Vì vậy, chúng tôi muốn tạo ra một dòng trong mỗi thư mục mà chúng tôi muốn chia sẻ. Vì trong ví dụ này client có địa chỉ 192.168.56.22, các dòng của chúng tôi sẽ như sau:

/home		192.168.56.22(rw,sync,no_root_squash,nosubtree_check)
/var/nfs	192.168.56.22(rw,sync,nosubtree_check)

Chúng tôi đã giải thích mọi thứ ở đây:
rw: Tùy chọn này cho phép client đọc và ghi vào ổ đĩa.

sync: Tùy chọn này buộc NFS phải ghi các thay đổi vào đĩa trước khi trả lời. Điều này dẫn đến môi trường ổn định và nhất quán, vì phản hồi phản ánh tình trạng thực tế của một volume từ xa.

nosubtreecheck: Tùy chọn này ngăn việc kiểm tra cây con, đó là một tiến trình mà máy chủ host phải kiểm tra xem tệp có thực sự vẫn còn trong export tree mỗi khi có yêu cầu. Điều này có thể có nhiều vấn đề khi một tập tin được đổi tên trong khi client được mở. Trong hầu hết trường hợp, tốt hơn là vô hiệu hóa việc kiểm tra subtree.

norootsquash: Mặc định, NFS dịch các yêu cầu từ người dùng gốc từ xa sang người dùng không có quyền trên máy chủ. Điều này được cho là một tính năng bảo mật bằng cách không ch phép một tài khoản root trên máy client sử dụng hệ thống tập tin của máy chủ host như là người dùng root. Chỉ thị này vô hiệu hóa một số chia sẻ nhất định.

 Khi bạn hoàn tất thay đổi, lưu và đóng tập tin.

Tiếp theo bạn sẽ tạo các bảng NFS chia sẻ bằng các gõ:

root@server:~# exportfs -a

Tuy nhiên, dịch vụ NFS chưa chạy. Để chạy nó bạn gõ như sau:

root@server:~# service nfs-kernel-server start

Điều này sẽ làm cho việc chia sẻ sẵn sàng cho các client mà bạn đã cấu hình.

Tạo các điểm gắn và gắn các chia sẻ từ xa trên các máy chủ Client.

Bây giờ, máy chủ host được cấu hình và thực hiện chia sẻ các thư mục đã sẵn sàng, bạn cần chuẩn bị cho client.

Chúng tôi sẽ phải gắn các thư mục chia sẻ từ xa, vì vậy chúng ta hãy tạo ra các điểm để mount. Chúng tôi sử dụng thư mục truyền thống /mnt làm điểm bắt đầu và tạo một thư mục gọi là nfs ở dưới nó để chia sẻ các thư mục của họ.

Các thư mục thực tế sẽ tương ứng với vị trí của chúng tôi trên máy chủ host. Chúng ta có thể tạ ra mỗi thư mục và các thư mục cha cần thiết, bằng các gõ như sau:

root@client:~# mkdir -p /mnt/nfs/home
root@client:~# mkdir -p /mnt/nfs/var/nfs

Bây giờ chúng ta có một chỗ để chia sẻ từ xa của chúng ta, chúng ta có thể gắn chúng bằng các địa chỉ của các máy chủ host, trong hướng dẫn này là 192.168.56.21:

root@client:~# mount 192.168.56.21:/home /mnt/nfs/home
root@client:~# mount 192.168.56.21:/var/nfs /mnt/nfs/var/nfs

Chúng sẽ gắn các thư mục chia sẻ từ máy chủ host đến các máy client. Chúng ta có thể kiểm tra lại bằng cách nhìn vào không gian trên đĩa có sẵn trên máy chủ client của chúng tôi

root@client:~# df -h

Filesystem              Size  Used Avail Use% Mounted on
udev                    485M  4.0K  485M   1% /dev
tmpfs                   100M  504K   99M   1% /run
/dev/dm-0               5.6G  1.8G  3.5G  34% /
none                    4.0K     0  4.0K   0% /sys/fs/cgroup
none                    5.0M     0  5.0M   0% /run/lock
none                    497M     0  497M   0% /run/shm
none                    100M     0  100M   0% /run/user
/dev/sda1               236M   41M  183M  19% /boot
192.168.56.21:/home     5.6G  1.8G  3.5G  34% /mnt/nfs/home
192.168.56.21:/var/nfs  5.6G  1.8G  3.5G  34% /mnt/nfs/var/nfs

Như bạn thấy ở cuối, chỉ có một trong số các thư mục chia sẻ đã xuấ hiện. điều này là bởi cả hai thư mục chia sẻ đều nằm trên cùng một hệ thống tập tin trên máy chủ từ xa, có nghĩa là chúng chia sẻ cùng một bộ lưu trữ. Để các cột Avail và Use% vẫn chính xác, chỉ có thể thêm một phần vào các phép tính.

Nếu bạn muốn xem tất cả các thư mục chia sẻ NFS mà bạn đã gắn, bạn có thể nhập:

root@client:~# mount -t nfs

192.168.56.21:/home on /mnt/nfs/home type nfs (rw,vers=4,addr=192.168.56.21,clientaddr=192.168.56.22)
192.168.56.21:/var/nfs on /mnt/nfs/var/nfs type nfs (rw,vers=4,addr=192.168.56.21,clientaddr=192.168.56.22)

Thao tác này sẽ hiển thị tất cả các gắn NFS hiện có truy cập trên máy client của bạn.

###Kiểm tra truy cập NFS

Bạn có thể kiểm tra quyền truy cập vào thư mục chia sẻ của bạn bằng cách viết một cái gì đó vào thư mục của bạn. Bạn có thể viết tập kiểm tra vào trong các thư mục như sau:

root@client:~# touch /mnt/nfs/home/test_home

Hãy viết tập tin kiểm tra vào phần chia sẻ khác để chứng minh một sự khác biệt quan trọng.

root@client:~# touch /mnt/nfs/var/nfs/test_var_nfs

Nhìn vào quyền sở hữu của tập tin trong thư mục home:

root@client:~# ls -l /mnt/nfs/home/test_home

-rw-r--r-- 1 root   root      0 Apr 30 14:43 test_home


Như bạn thấy, tập tin này thuộc sở hữu của root. Điều này là do chúng ta đã vô hiệu hóa tùy chọn root_squash trên phần gắn kết này mà có thể đã viết tệp như là một người vô danh, không phải là root.

Trên tập tin kiểm tra khác của chúng tôi, được gắn kết với root_squash kích hoạt, chúng ta sẽ thấy một cái gì đó khác nhau:

root@client:~# ls -l /mnt/nfs/var/nfs/test_var_nfs

-rw-r--r-- 1 nobody nogroup 0 Apr 30 14:44 test_var_nfs

Như bạn thấy, tệp này đã được gán cho người dùng 'nobody' và nhóm 'nogroup'. Điều này theo cấu hình của chúng tôi

###Thực hiện gắn thư mục NFS từ xa từ động

Chúng ta có thể thực hiện việc gắn các thư mục NFS từ xa tự động bằng cách thêm nó vào tệp fstab của chúng tôi trên máy client.

Mở tập tin này với các đặc quyền root với các trình soạn thảo văn bản:

root@client:~# nano /etc/fstab
Ở cuối tệp, chúng tôi sẽ thêm một dòng cho mỗi thư mục chia sẻ của chúng tôi. Chúng sẽ giống như sau:

192.168.56.21:/home    /mnt/nfs/home   nfs auto,noatime,nolock,bg,nfsvers=4,intr,tcp,actimeo=1800 0 0
192.168.56.21:/var/nfs    /mnt/nfs/var/nfs   nfs auto,noatime,nolock,bg,nfsvers=4,sec=krb5p,intr,tcp,actimeo=1800 0 0

Các tùy chọn mà chúng ta chỉ định ở đây có thể được tìm thấy trong trang man của mô tả gắn NFS trong tập tin fstab

root@client:~# man nfs

Điều này sẽ tự động gắn các phân vùng từ xa khi khởi động (có thể mất vài phút để kết nối được thực hiện và chia sẻ có sẵn).

###Ngừng gắn chia sẻ thư mục NFS từ xa

Nếu bạn không còn muốn gắn thư mục từ xa được gắn kết trong hệ thống của mình, bạn có thể thao gắn kết dễ dàng bằng cách di chuyển ra khỏi cấu trúc thư mục chia sẻ và ngắt kết nối, như sau:

root@client:~# cd ~
root@client:~# unmont /mnt/nfs/home
root@client:~# unmount /mnt/nfs/var/nfs

Điều này sẽ xóa bỏ các thư mục chia sẻ từ xa, chỉ để lại bộ nhớ cụ bộ của bạn có thể truy cập được:

root@client:~# df -h

Filesystem      Size  Used Avail Use% Mounted on
/dev/vda         59G  1.3G   55G   3% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
udev            2.0G   12K  2.0G   1% /dev
tmpfs           396M  320K  396M   1% /run
none            5.0M     0  5.0M   0% /run/lock
none            2.0G     0  2.0G   0% /run/shm
none            100M     0  100M   0% /run/user
Như bạn thấy, thư mục chia sẻ NFS của chúng tôi không còn tồn tại trong không gian lưu trữ nữa.
