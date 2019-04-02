## Mục lục

------
#### [1 Giới thiệu](#1)
   - [1.1 Thành Phần](#1.1)
#### [2 Cài Đặt](#2)

   - [2.1 Cài đặt packet](#2.1)
   - [2.2 Cài Elasticsrach](#2.2)
   - [2.3 Cài đặt Mongodb](#2.3)
   - [2.4 Cài đặt Graylog](#2.4)
   - [2.5 Cầu hình web interface](#2.5)
   - [2.6 Cầu hình Rsyslog](#2.6)
   - [2.7 Web GUI](#2.7)
   - [2.8 Alert Mail (đang update)](#2.8)
#### [3 Tham khảo](#3)
   - [3.1 Liên hệ](#3.1)
   


-------------------


### <a name="1"><a/>1 Giới thiệu.

- Graylog là một nền tảng quản lí log mã nguồn mở với nhiều tính năng mạnh mẽ. Nó có khả năng gộp chung và giải nén các dữ liệu quan trọng từ server log, thứ thường được gửi sử dụng giao thức Syslog. Bên cạnh đó Graylog cũng hỗ trợ việc tìm kiếm và giúp bạn hình dung cấu trúc của log thông qua một giao diện web.

- Bài viết này sẽ hướng dẫn bạn cài đặt và cấu hình Graylog trên server **Centos7**, đồng thời thiết lập một đầu vào đơn giản dùng để ghi system log.

### <a name="1.1"><a/>1.1 Thành Phần.

   - **MongoDB**: Lưu trữ các cấu hình và thông tin meta
   
   - **Elasticsearch**: Lữu trữ log và cung cấp dữ liệu tìm kiếm, phần chiếm nhiều tài nguyên các các hành động phần lớn sẽ được thự hiện tại đây.
   
   -  **GrayLog**: Phân tích cú pháp log và thu thập log từ các client gửi lên.
   
   - **GrayLog Web interface**: Quản lý bằng giao diện web
   
   
### <a name="2"><a/>2 Cài đặt 

- Tắt vào firewalld

        systemctl stop firewalld
        
        systemctl disable firewalld
        
        sudo setenforce 0
       
        reboot  
   

### <a name="2.1"><a/>2.1 Cài đặt Packet.

        
                   
        yum -y install epel-release
        
        
   - Install java 
   
        - Elasticsearch dựa trên nên java nên cần cài đặt JDK
        
                yum install java
                
       ![image](https://user-images.githubusercontent.com/19284401/55300780-6e44aa00-5463-11e9-97bd-2f4ca91521ec.png)
       
       
### <a name="2.2"><a/>2.2 Cài đặt Elasticsearch 

 - Import the GPG key
 
        rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch   
        
 - Tạo repo mới cho **Elasticseach** 
 
         vi /etc/yum.repos.d/elasticsearch.repo  
         
 - Pates nội dung sau vào file
 
 
        [elasticsearch-2.x]
        name=Elasticsearch repository for 2.x packages
        baseurl=https://packages.elastic.co/elasticsearch/2.x/centos
        gpgcheck=1
        gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
        enabled=1
        
        
   - Hiện tại **Elasticseach**  phiên bản mới nhất là 6.7.
   
   - Tuy nhiên ở đây mình dùng bản 2.x, cụ thể hơn là 2.4. Vì đơn giản là bản 6.7 nó ngốn quá nhiều tài nguyện luôn.
   
   ![image](https://user-images.githubusercontent.com/19284401/55300976-7ea95480-5464-11e9-8316-75fe7bafa291.png)
   
   - Với cấu hình như trên mình chạy bản 5.x mà được vài phút là **Elasticseach** lăn ra chết vì ko đủ RAM. nhưng với bản 2.4 thì chạy gần 1 năm nay ko bị sao cả.
   
 - Cài đặt elasticsearch

       yum -y install elasticsearch
 
 
 - Cấu hình elasticsearch
 
        systemctl daemon-reload 
        
        systemctl enable elasticsearch.service
        
- Cấu hình elasticsearch

        vi /etc/elasticsearch/elasticsearch.yml        

     - Tìm đến dòng 17 sửa thành **graylog**
     
     ![image](https://user-images.githubusercontent.com/19284401/55301366-7a7e3680-5466-11e9-8628-becbb3463901.png)
     
 - Restart elasticsearch
 
        systemctl restart elasticsearch.service   
        
        
 - Check lại thông tin elasticsearch
        
        curl -X GET http://localhost:9200
        
   ![image](https://user-images.githubusercontent.com/19284401/55301675-edd47800-5467-11e9-900d-a4a10c75afd5.png)
   
 - Kiểm tra tình trạng "sức khỏe" của  elasticsearch 
 
        curl -XGET 'http://localhost:9200/_cluster/health?pretty=true'
        
        
   - Nếu bạn thấy dòng **"status" : "green"** là ngon rồi. 
   
     ![image](https://user-images.githubusercontent.com/19284401/55301816-9c78b880-5468-11e9-80ac-2fc39db144c8.png)
     
     
     
### <a name="2.3"><a/>2.3 Cài đặt Mongodb  

- Creat repo

        vi /etc/yum.repos.d/mongodb-org-3.6.repo
        
- Pates nội dung sau vào file 
        
        [mongodb-org-3.0]
        name=MongoDB Repository
        baseurl=http://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.0/x86_64/
        gpgcheck=0
        enabled=1
        
        
- Cài đặt Mongodb

          yum -y install mongodb-org 
        
- Nếu firewalldđang bật

        yum -y install policycoreutils-python
        
        semanage port -a -t mongod_port_t -p tcp 27017  
        
-  Nếu bạn ko muốn dùng firewalld
        
       systemctl stop firewalld
    
       systemctl disable firewalld
       
- Start và enable Mongodb

        systemctl start mongod
        
        systemctl enable mongod
        
### <a name="2.4"><a/>2.4 Cài đặt Graylog

- Tạo repo
            
            vi /etc/yum.repos.d/graylog.repo

- Pates nội dung sau vào file.

            [graylog]
            name=graylog
            baseurl=https://packages.graylog2.org/repo/el/stable/2.4/$basearch/
            gpgcheck=1
            repo_gpgcheck=0
            gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-graylog    
            
            
- Cài đặt

            yum -y install graylog-server
            
- Cấu hình 

            vi /etc/graylog/server/server.conf
            
- Tìm đến dòng 55 Thêm bất kì những ký tự nào đó trên bàn phím vào sau dâu = cũng được (có có ký tự đặc biệt)

    password_secret =fahiogbfaerogiboierahgowebgaw0eoigbwguoptgbobofrbwepiufbgierubgfwuipfbw
    
    ![image](https://user-images.githubusercontent.com/19284401/55302346-ec587f00-546a-11e9-8959-c8e05b442cf2.png)
    
- Nếu bạn muộn thể hiện sự chuyên nghiệp thì bạn dùng lệnh sau để gen ra 1 chuỗi ngẫu nhiên rồi pates sau dấu =

            yum -y install pwgen
            
            pwgen -N 1 -s 96
            
- Tìm tiếp đến dòng 66 thay đoạn code vào sau dâu = 

        echo -n yourpassword | sha256sum 
        
  - Thay  **yourpassword** bằng password của bạn
        
  ![image](https://user-images.githubusercontent.com/19284401/55302715-a00e3e80-546c-11e9-9f63-995825d93268.png)
        
  - Chú ý bỏ cài dấu - ở cuối dùng đi nhé 
   
  ![image](https://user-images.githubusercontent.com/19284401/55302734-b9af8600-546c-11e9-977a-9668ea12ac5b.png)
  
  
### <a name="2.5"><a/>2.5 Cấu hình Graylog web interface
        
        vi /etc/graylog/server/server.conf
        
-  Tìm đến dòng 81 và 132 vào sửa lại ip cho hợp lý.        
            
   ![image](https://user-images.githubusercontent.com/19284401/55303068-56beee80-546e-11e9-9e2c-4f88dd60b1a3.png)
   
   ![image](https://user-images.githubusercontent.com/19284401/55303124-92f24f00-546e-11e9-8121-2039203481b9.png)
   
   
- Restart và enable graylog-server
 
        systemctl daemon-reload
        
        systemctl restart graylog-server
        
        systemctl enable graylog-server


- Graylog hỗ trợ khá nhiều cách để có thể nhận được log từ client gửi lên. Tuy nhiên mình sẽ hướng dẫn mọi người các đơn giản nhất đó là dùng **rsyslog**

### <a name="2.6"><a/>2.6 Cấu hình rsyslog.

     
   - Install rsyslog
   
            yum install rsyslog
            
   - Cấu hình.
   
            vi /etc/rsyslog.conf
            
   - Để có thể gửi log lên server bạn cần chỉnh sửa lại file 
   
   -  Thêm đoạn code này vào  file
   
            $ModLoad imfile
            
   -  Cú pháp gửi log lên server
   
            $InputFileName /var/log/httpd/xxxx_access.log
            $InputFileTag xxxAccessLog:
            $InputFileStateFile xxxx_acc.log.statefile
            $InputFileFacility local3
            $InputFileSeverity info
            $InputRunFileMonitor
            
            
   - InputFileName: Đường dẫn đến file log 
   - InputFileTag: Thẻ tab (đặt tên gì cũng được cốt là để dễ nhớ)
   - InputFileFacility 
   
   ### Tham khảo thêm về $ModLoad imfile  <a href="https://www.rsyslog.com/doc/v5-stable/configuration/modules/imfile.html" rel="nofollow">tại đây<a/>
   
   ### Tham khảo về Facility và Severity level <a href="https://success.trendmicro.com/solution/TP000086250-What-are-Syslog-Facilities-and-Levels" rel="nofollow">tại đây<a/>
   
- Trỏ về server log
   
         local3.* @10.0.1.129:9999
        
   - **local3** là Facility ở trên
   
   - @10.0.1.129: là ip của server graylog
   
   - 9999: là port của graylog
   
- Chú ý: Nếu firewalld đang mở nhớ mở port nhé
   
 
 
#### <a name="2.7"><a/>2.7 Web GUI  

- Truy cập 

    http://your_ip_server:9000
    
    ![image](https://user-images.githubusercontent.com/19284401/55304682-d486f800-5476-11e9-963c-1c54c967c277.png)
    
    - user mặc định là **admin** 
    
    - pass bạn đã tạo ở bước trước "echo -n **yourpassword** | sha256sum "
    
- Đăng nhập xong bạn vào tab **SEARCH**

    ![image](https://user-images.githubusercontent.com/19284401/55304827-94744500-5477-11e9-97a7-269180ffbbc3.png)
    
    - Đây là nơi hiển thị toàn bộ log mà server thư thập được từ client gửi lên.
    
- Tab **Stream**
        
    ![image](https://user-images.githubusercontent.com/19284401/55315062-af55b200-5495-11e9-8972-71a5610b9910.png)

    
   - Đây là nơi các bạn phân loại logs(access, error, warning...) để dễ quản lý.
   
   - Tạo Stream mới, góc trên bên phải.
   
   ![image](https://user-images.githubusercontent.com/19284401/55315199-06f41d80-5496-11e9-8716-c67b9f82de58.png)
   
   - Tạo Rules cho Stream.
   
   
   - Nhìn lại tab Search 1 chút các bạn sẽ thấy nhưng thông tin này ở đó.
      
   ![image](https://user-images.githubusercontent.com/19284401/55315973-a534b300-5497-11e9-89c3-60c44e5b778a.png)
   
   
   - Phần Field và phần value sẽ là điều kiện để ghi nhận log vào stream 
   
   ![image](https://user-images.githubusercontent.com/19284401/55315738-25a6e400-5497-11e9-95b2-3287f0d3ff07.png)
   
   - Như trong VD dưới đây Field mình chọn message và Value mình chọn WowzaError
   
   ![image](https://user-images.githubusercontent.com/19284401/55375179-71a86600-5535-11e9-921b-ca1f46d97e36.png)
   
   - Tức là khi có log gửi lênh. Graylog sẽ check điều kiện để gắn message này vào stream nào
   
   ![image](https://user-images.githubusercontent.com/19284401/55375329-f1363500-5535-11e9-9bf7-4508ca385bdf.png)
   
   - Như Các bạn thấy trong Stream này mình có 2 điều kiện
    
     - 1 Field facility must match regular expression local4
     
     - 2 Field message must match regular expression WowzaError
     
   -  Nếu đủ 2 điều kiện này thì sẽ cho log này vào stream.
   
   ![image](https://user-images.githubusercontent.com/19284401/55375463-70c40400-5536-11e9-9c2b-31b8d5d1d1fd.png)

   
   ![image](https://user-images.githubusercontent.com/19284401/55315491-9ef20700-5496-11e9-9849-86bc3a6d2199.png)

   
   ![image](https://user-images.githubusercontent.com/19284401/55315616-de205800-5496-11e9-9789-893dd4bd5c2f.png)
   
   - Tab Alert này sẽ liên quan để phần cấu hình mail, mình sẽ nói cuối cùng
   
   ![image](https://user-images.githubusercontent.com/19284401/55316387-7b2fc080-5498-11e9-9b07-2be078ab899d.png)
   
   - Tab System -> input
   
   ![image](https://user-images.githubusercontent.com/19284401/55341914-be5a5580-54d1-11e9-86d6-caf9a2fe9ecb.png)


   - Tab này là nơi cấu hình để client gửi log lên.
   
   ![image](https://user-images.githubusercontent.com/19284401/55342004-f6619880-54d1-11e9-80e6-716b7d86104e.png)
   
   - Bạn chọn **Raw/plantext UDP** hoặc **syslog UDP** rồi **LAUNCH NEW INPUT** ngay bên cạnh 
   
   - Điểm khác này của 2 cái này là gì.
    
   - Syslog UDP: đơn giản là gửi theo cấu trúc của Rsyslog thôi.
   
   - Raw/plantext UDP: Cài này các bạn chú ý này. nếu **client và graylog ko cùng timelocal**, thì các bạn phải dùng option này thì mới xem được log, nếu ko chọn cái này thì graylog vẫn ghi nhận có log nhưng nó ko hiện thị ra được vì giờ giữa client và nó ko đồng bộ được.
   
   ##### Syslog UDP
   
   ![image](https://user-images.githubusercontent.com/19284401/55342912-1a25de00-54d4-11e9-921b-dd5646f3c94d.png)

   
   - Global: nếu bạn có nhiều client gửi log lên server bằng cơ chế Rsyslog.
   
   - Title: Đặt tên thôi.
   
   - Bin Address: Các bạn để 0.0.0.0 là cho phép tất cả các client gửi log lên.
   
   - port: client sẽ gửi log qua port này. đã nói ở phần cấu hình **Rsyslog**
   
   - Store Full Message: lưu lại toàn bộ log ở dạng full message mà ko xóa.
   
   
   ##### Raw/plantext UDP:
   
   ![image](https://user-images.githubusercontent.com/19284401/55342975-3c1f6080-54d4-11e9-9238-87883a3c25bf.png)

   - Khá giống với **Syslog UDP**
   
    
- Nếu đơn giản chỉ là quản lý log tập trung thì có rất nhiều các tool khác, chả cần phải phục tạp thế này. Nhưng cái hay cảu Graylog là nó tích hợp sẵn mail cho người dùng, chỉ cần cấu hình là xong.   
   
##### <a name="2.8"><a/>2.8 Cấu hình Alert mail

- Cấu hình lại file config 
        
              vi /etc/graylog/server/server.conf
              
-  Tìm đến dòng 497  **# Email transport** Đây là phần cấu hình liên quan đến gửi mail


    # Email transport
    498 #transport_email_enabled = false
    499 #transport_email_hostname = mail.example.com
    500 #transport_email_port = 587
    501 #transport_email_use_auth = true
    502 #transport_email_use_tls = true
    503 #transport_email_use_ssl = true
    504 #transport_email_auth_username = you@example.com
    505 #transport_email_auth_password = secret
    506 #transport_email_subject_prefix = [graylog]
    507 #transport_email_from_email = graylog@example.com

 
 - transport_email_enabled: cho phép gửi mail
 
 - transport_email_hostname: server mail  
  
 - transport_email_port: port
  
 - transport_email_use_auth = true
 - transport_email_use_tls = true
 - transport_email_use_ssl = true        bật xác thực

 - transport_email_auth_username: địa chỉ mail
 
 - transport_email_auth_password: password mail
 
 - transport_email_from_email: nhập lại địa chỉ mail.
 
 - transport_email_web_interface_url: nhập ip và port của server (nếu các bạn public graylog ra ngoài thì phải có dòng này nhé).
 
 
![image](https://user-images.githubusercontent.com/19284401/55372033-6bf95300-552a-11e9-8a81-fceeaae083ab.png)

Tham khảo thêm <a href="https://community.graylog.org/t/setting-up-email-transport-config/5445" rel="nofollow"> tại đây.<a/>


- Restart lại graylog 

        systemctl daemon-reload
        
        systemctl restart graylog-server

- Các bạn ko nên chỉnh sửa trực tiếp vào mẫu, mà nên copy ra để sửa tránh trường hợp cấu hình sai, lại ko biết sửa lại thể nào.

- Chú ý: ko nếu dùng Gmail hoặc Gsuite phần cho phép tài khoản đăng nhập từ ứng dụng kém an toàn <a href="https://myaccount.google.com/lesssecureapps" rel="nofollow"> tại đây
<a/>

- Quay lại giao diện web

- Alert -> Manage conditions

    ![image](https://user-images.githubusercontent.com/19284401/55372795-238f6480-552d-11e9-9a0a-a216279e3612.png)

   - Add New Conditions **Alert on stream** và **Condition Type** -> Add Alert Condition
        
    ![image](https://user-images.githubusercontent.com/19284401/55372872-60f3f200-552d-11e9-98e8-ff8137f8b4d8.png)
    ![image](https://user-images.githubusercontent.com/19284401/55372906-8254de00-552d-11e9-980c-ac618640cab3.png)
     
     - Alert on stream: ở đây sẽ list ra toàn bộ các stream mà bạn đã tạo. bạn muốn nhận mail thông báo khi stream nào thì bạn chọn stream đó.
     
     - Condition Type: Điểu kiện để gửi mail 
     
     
  - Alert -> Manage notification  -> Add new notification
  
   ![image](https://user-images.githubusercontent.com/19284401/55373405-31de8000-552f-11e9-8c87-8863088b1889.png)
   ![image](https://user-images.githubusercontent.com/19284401/55373445-563a5c80-552f-11e9-9eeb-bbf5a6760986.png)

  - Mục Notification -> Add Alert notification

   ![image](https://user-images.githubusercontent.com/19284401/55373600-e5e00b00-552f-11e9-982b-2825d352bdcf.png)
   
   - notifi on stream: các bạn chọn đúng cái stream mà các bạn đã chọn ở mục  **Add Alert Condition** 
   
   - Notification Type: các bạn chọn kiểu Email Alert Callback để gọi đến hàm gửi mail
   
   
  - sau khi **Add Alert notification** bạn sẽ có 1 popup như hình dưới
   
   ![image](https://user-images.githubusercontent.com/19284401/55373865-bb428200-5530-11e9-9c3b-206e561585f6.png)
   
   - Title: Đặt tên cho notifications
   - Sender: Nhập địa chỉ email đã cấu hình trong file config ở trên  (transport_email_auth_username = you@example.com)
   - Email receivers: nhập email nhập mail thông báo (nhập xong 1 email các bạn tab 1 cái để nhập emmail tiếp theo).
   
  ![image](https://user-images.githubusercontent.com/19284401/55374207-cfd34a00-5531-11e9-9eac-bbe631169a53.png)

  Cuối cùng là **Save**
  
- Giờ chúng ta cùng test alert mail.

![image](https://user-images.githubusercontent.com/19284401/55374771-e4b0dd00-5533-11e9-8732-dbd1c371d01a.png)

  - Mình echo 1 chuỗi vào file log để server ghi nhận.
  
  - Kiểm tra lại trong phần Stream xem đã ghi nhận log chưa.
  
  ![image](https://user-images.githubusercontent.com/19284401/55374893-58eb8080-5534-11e9-861c-24220178a8e9.png)
  
   - Giờ ngồi đợi mail.
   
  ![image](https://user-images.githubusercontent.com/19284401/55375055-ddd69a00-5534-11e9-9417-45bd1837b45e.png)
  
  - Sau khoảng 1 phút thì đã nhận được mail của graylog rồi. Xử lý cũng khá nhanh.
  
Như vậy mình đã hướng dẫn xong Graylog server. Chúc các bạn thành công.

#### <a name="3"><a/> 3. Tài liệu tham khảo.

1. Cài đặt Graylog http://docs.graylog.org/en/3.0/pages/installation/os/centos.html

2. Cài đặt Mongodb https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/

3. Cài đặt Elasticsearch https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-elasticsearch-on-centos-7


#### <a name="3.1"> 3.1 Liên hệ.

<a href="https://www.facebook.com/trunglv.91" rel="nofollow">Facebook<a>


  
   
   
   

   

    
    

    
    
 
   
    
    

            
   
   

   
   
   

   
   



   
   
   
   

    
    

    
    
  
  
 
 
      
                     
                             
        
        
               
     
     
   
 
 
   

     
     
     


