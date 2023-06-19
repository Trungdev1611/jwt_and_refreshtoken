#Các bước của project
1. Người dùng login bên phía client, server xác minh username, password sau đó
sẽ trả ra cho client data có chứa accesstoken và thực hiện set refreshtoken vào cookie bên phía server . refreshtoken set trong cookie sẽ có thời gian sống dài hơn nhiều accesstoken gửi cho phía client

 res.cookie('refreshtokenIncookie', reFreshToken, {  sameSite: 'none',secure:true,
    //  secure: true, cái này cho https
     maxAge: 24 * 60 * 60 * 1000 ,  httpOnly : true});
2. Client sẽ nhận được accesstoken trong data trả về từ phía server và cookie trong header reponse từ phía server, client sẽ lưu accesstoken trong localStorage và cài đặt thêm thư viện jwt deccode phía client để có thể biết được thời gian hết hạn của accesstoken,. cookie của server gửi chỉ xuất hiện trong header response của rlogin request mà không thể nhìn thấy trong phần cookie của trình duyệt cũng như không thể truy cập được bằng mã js bên phía client. cookie này sẽ tự động được gửi lên server trong các request sau đó. Ta có thể truy cập cookie này phía server bằng lệnh 
req.cookie.refreshtokenIncookie
const refreshtoken = refreshtokenCookie?.split("=")?.[1]

3. Sau khi login, mỗi khi client định gửi 1 request lên server, trong interceptor của axios sẽ có hàm chekc xem accesstoken có tồn tại hay không và nếu tồn tại thì accesstoken đã hết hạn hay chưa bằng cách so sánh nó với thời gian hiện tại, nếu accesstoken hết hạn request trước đó sẽ không được thự hiện ngay mà sẽ gọi lên 1 endpoint refreshtoken để lấy 1 token mới, khi token mới trả về thì mới tiếp tục thực hiện request trước đó

4. endpoint refreshtoken sẽ nhận được refreshtoken trong cookie và tiến hành verify bình thường như accesstoken, khi verify thành công, server sẽ tạo 1 accesstoken mới để trả về client, client dùng accesstoken mới này lưu lại vào localStorage để tiếp tục thực hiện các yêu cầu khác

5. Khi người dùng logout, ta cần có 1 enpoint logout trên phía server để client gọi lên, phía server sẽ thực hiện clear cookie refreshtoken