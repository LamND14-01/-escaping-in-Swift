<h1>What is @escaping in Swift?</h1>

Nếu bạn từng viết một function có argument là 1 closure, bạn có thể đã từng sử thấy qua keyword `@escaping`. Khi một closure dược dánh dấu là escapeing, có nghĩa là closure đó tồn tại lâu hơn, hoặc vượt khỏi phạm vi mà bạn đã chuyển nó cho. Đây là ví dụ của 1 non-escaping closure:

```Swift
    func doSomething(using closure: () -> Void) {
      closure()
    }
```   

The closure truyền cho `doSomething(using:)` được thực thi ngay lập tức trong `doSomething(using:)` function. Bởi vì closure được thực hiện ngay lập tức trong function `doSomething(using:)` nên ta có thể biết được không có gì của closure có thể leak hoặc tồn tại lâu hơn `soSomething(using:)`. Nếu ta thực hiện closure trong `doSomething(using:)` tồn tại lâu hơn hoặc vượt khỏi phạm vi của function, trình biên dịch sẽ báo lỗi:

```Swift
    func doSomething(using closure: () -> Void) {
      DispatchQueue.main.async {
        closure()
      }
    }
```    

Ví dụ trên gây ra compiler error vì Swift thấy rằng khi ta gọi `doSomething(using:)`, the closure được truyền vào sẽ vượt khỏi phạm vi của nó. Điều này có nghĩa là ta cần đánh dấu điều này là cố ý để `doSomething(using:)` sẽ biết rằng họ đang xử lý một closure sẽ vượt khỏi phạm vi của hàm, điều đó cũng có nghĩa là ta cần phải đề phòng một số vấn đề kèm theo ví dụ như `capturing self weakly`. Ngoài việc thông báo cho `doSomething(using:)` về escaping closure, nó cũng báo cho Swift compiler rằng ta biết closure vượt ngoài phạm vi mà nó được truyền đến và ta chấp nhận việc đó.

Bạn sẽ thường thấy escaping closures dành cho các function xử lý bất đồng bộ và gọi closure như một callback. Ví dụ, `URLSession.dataTask(with:completionHandler:)` có một `completionHandler` được đánh dấu `@escaping` vì closure được truyền vào như một completion handler được thực hiện khi request completes, đó là 1 thời điểm sau khi data task được khởi tạo. Nếu bạn viết code có một completion handler và sử dụng data task, closure bạn truyền vào phải được đánh dấu là `@escaping`:

```Swift
    func makeRequest(_ completion: @escaping (Result<(Data, URLResponse), Error>) -> Void) {
      URLSession.shared.dataTask(with: URL(string: "https://donnywals.com")!) { data, response, error in
        if let error = error {
          completion(.failure(error))
        } else if let data = data, let response = response {
          completion(.success((data, response)))
        }
    
        assertionFailure("We should either have an error or data + response.")
      }
    }
```    

Lưu ý rằng trong mã code trên `completion` closure được đánh dấu là `@escaping`. Nó phải có vì ta sử dụng trong data task's completion handler. Điều đó có nghĩa `completion` closure không được thực thi cho đến khi `makeRequest(_:)` đã xử lý xong và closure sẽ vượt ngoài phạm vi của nó.

Nói ngắn gọn, `@escaping` được sử dụng để thông báo cho function về closure của nó có thể được lưu trữ hoặc vượt ngoài phạm vi của function. Điều này nghĩa là người gọi cần đề phòng một số lỗi liên quan tới retain cycles và memory leaks. Nó cũng báo cho Swift compiler biết đây là cố ý.