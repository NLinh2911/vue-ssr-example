# So sánh công nghệ Sever-Side Rendering

Xây dựng SSR web cho service chứa nhiều thông tin, nội dung không tùy biến theo từng người dùng và cần cho SEO. Ví dụ, blog service có thể là 1 SSR service riêng biệt trong khi phần web app học online là client-side rendering web application tùy biến theo dữ liệu của người dùng - học viên. 

## Vấn đề với Vue thuần:
* Vue app gốc là client-side rendering nên nếu xem phần sources thì trang HTML không chứa đầy đủ các dữ liệu (innerHTML content). Dữ liệu đc hiển thị khi chạy Vue JS code.
* Trong khi SSR webpages chứa đầy đủ innerHTML, dữ liệu đc trả về hết trước khi hiển thị hoàn chỉnh cho client nên SEO dễ dàng hơn

## Cấu hình Vue SSR 

### Sử dụng plugin để Prerender: `prerenderer-spa-plugin`

Prerendering là 1 giải pháp để hiển thị trang web đầy đủ nội dung nhằm nâng cao SEO. Thay vì SSR thì ở build time của Vue app, plugin trên sẽ tạo ra các static webpages

Với cấu hình dưới đây, lúc build Vue app, các trang ở routes `/`, `/about`, `/contact` sẽ đc chuyển thành các static webpages sẵn. 
```js
// webpack.conf.js
var path = require('path')
var PrerenderSpaPlugin = require('prerender-spa-plugin')

module.exports = {
  // ...
  plugins: [
    new PrerenderSpaPlugin(
      // Absolute path to compiled SPA
      path.join(__dirname, '../dist'),
      // List of routes to prerender
      [ '/', '/about', '/contact' ]
    )
  ]
}
```

* **Điểm cộng:**
  * Cấu hình nhanh, ít, đơn giản hơn so với SSR
* **Điểm trừ:**
  * Không phù hợp nếu có quá nhiều trang cần prerender (hàng trăm, ngàn trang)
  * Dữ liệu thay đổi thường xuyên: Có thể cấu hình web host để tự động tạo lại các static webs trong 1 khoảng thời gian nhất định để cập nhật dữ liệu (chưa biết cách - cần tìm hiểu thêm)
### Sử dụng Vue SSR:

Với Vue SSR, sử dụng module `vue-server-renderer`

* **Điểm cộng:**
  * Customizable cao, cấu hình hoàn chỉnh tùy biến theo ý mình
* **Điểm trừ:**
  * Cấu hình khá phức tạp, dài
  * Tìm hiểu, code theo cần thời gian

### Xây dựng 1 app thuần server render kiểu với Express-Nunjucks

* **Điểm cộng:**
  * Nhanh, dễ dùng, đã quen thuộc
  * Chuẩn SSR, SEO dễ dàng
* **Điểm trừ:**
  * Tech stack trong service này (ví dụ blog service) sẽ khác các service khác (ví dụ course services tùy biến theo clients xây dựng bằng Vue)


### Ý kiến của em: 
Nếu mình đã xây dựng microservices, nhìn qua thì bọn em định chia thành vài services chính như:
* CMS: Blog service
* Online learning: Course service
* User service
* Accounting & Payments service 
  * Ý kiến em nghĩ nên tách Accounting và Payments thành 2 services riêng biệt. Payment là thanh toán khóa học, phụ thuộc nhiều vào third-party services như banks, visa,... Còn accounting nghiêng về tính toán lương thương, tài chính nội bộ, logic này sẽ không bị ảnh hưởng trực tiếp bởi những transactions của payments mà có thể chỉ nhận inputs là các số liệu tài chính.

SSR để SEO là cho blog và trang hiển thị khóa học. Blog service xây dựng riêng biệt, tech stack riêng biệt cũng đc. 


Còn trang hiển thị khóa học, nằm trong 1 SPA của course service thì trang liệt kê và thông tin chi tiết khóa học cần SSR nhưng những trang này nằm trong course service mà cái này tùy biến nội dụng học theo người dùng (sau khi đăng nhập) thì đang theo hướng client side Vue app. Không thể SSR và CSR trên cùng 1 SPA.

Có thể sử dụng Prerendering nếu tìm đc cách cấu hình web host build lại, không ảnh hưởng đến SPA course service là 1 client-side render Vue app. Nhưng dữ liệu cập nhật bị chậm và chưa rõ build lại thì có ảnh hưởng gì đến các file khác trong app không.
