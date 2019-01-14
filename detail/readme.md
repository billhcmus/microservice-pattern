# Microservice

## Giới thiệu

- Khi thiết kế một hệ thống sử dụng kiến trúc Microservices, ta phải đối mặt với các vấn đề phát sinh đi kèm với nó. Một số cách để mô tả và cải thiện thiết kế kiến trúc là sử dụng pattern language.
- Một pattern là một giải pháp có thể tái sử dụng để giải quyết một vấn đề trong một ngữ cảnh nhất định.
- Một pattern language là một tập hợp các pattern có liên quan với nhau, các pattern giải quyết các vấn đề nằm trong một domain nhất định.
- Ví dụ một số mẫu thiết kế như Strategy, Bridge trong lập trình hướng đối tượng, sẽ giải quyết một số vấn đề cụ thế. Tập hợp các mẫu nằm trong một domain như Structure Pattern giải quyết các vấn đề thuộc về kiến trúc của một lớp đối tượng.

## Microservices architechture pattern language

- Microservices architechture pattern language là một tập hợp các pattern giúp giải quyết các vấn đề phát sinh trong kiến trúc microservices.
- Pattern language trước hết giúp ta quyết định về việc áp dụng microservices hay không. Nó mô tả những ưu và nhược điểm của monolithic và microservice.

<p align="center">
    <img src="/detail/images/microservicepatternlanguage.jpg" alt="">
</p>

- Các pattern được chia ra làm 3 lớp riêng biệt:
  - Infrastructure patterns: Giải quyết vấn đề về infrastructure ngoài quá trình phát triển
  - Application infrastructrue: Một số vấn đề về infrastructure tác động đến development
  - Application patterns: Giải quyết vấn đề mà nhà phát triển đối mặt
- Các pattern được gom nhóm với nhau dựa trên loại vấn đề mà chúng giải quyết.
- Dưới đây là một vài pattern giải quyết vấn đề liên quan đến việc đảm bảo data consistency và query data.

## Data consistency patterns for implementing transaction management

- Để nới lỏng mối liên kết thì mỗi service sẽ sở hữu một database riêng. Tuy nhiên việc sở hữu riêng này lại xảy ra các vấn đề nghiêm trọng. Application phải đảm bảo tính consistency của data bằng cách sử dụng Saga Pattern thay vì tiếp cận cách truyền thống là sử dụng traditional distributed transaction như 2PC.

<p align="center">
    <img width="450" src="/detail/images/data-consistency-problem.png" alt="">
</p>

- Trong một service thì ACID Transaction vẫn có thể thực hiện được, tuy nhiên thách thức đặt ra là thực hiện transaction cho các operations thực hiện việc update data trên nhiều service.

- Tìm hiểu thêm về Saga pattern tại [đây](https://microservices.io/patterns/data/saga.html)

## Pattern for querying data in a microservice architecture

- Một vấn đề khác khi triển khai một database trên một service là khi truy vấn dữ liệu mà truy vấn cần phải kết hợp dữ liệu rải rác từ những service khác nhau. Dữ liệu của mỗi service chỉ được truy cập thông qua API, do đó không thể thực hiện distributed queries trên các database.

<p align="center">
    <img width="450" src="/detail/images/query-problem.png" alt="">
</p>

### Ví dụ về việc truy vấn dữ liệu trên nhiều service

- Giả sử rằng có một operation là findOrder(), operation này thực hiện việc truy vấn và lấy thông tin của một đơn đặt hàng thông qua orderId.

<p align="center">
    <img width="450" src="/detail/images/query-example.png" alt="">
</p>

- Với kiến trúc monolithic application thì việc truy vấn các thông tin trên chỉ cần qua một câu query SELECT và join một số bảng. Trong khi đó với microservice architecture dữ liệu được phân tán ở nhiều service khác nhau như:
  - Order Service
  - Kitchen Service
  - Accouting Service
  - Delivery Service
- Khi một client cần thông tin chi tiết về đơn hàng thì bắt buộc phải qua các service này.

### Giải pháp

- Thông thường ta có thể sử dụng Api Composition pattern, nó gọi những API của để lấy dữ liệu từ các service và tổng hợp chúng lại. Một số trường hợp khác ta phải sử dụng Command Query Responsibility Segregation pattern, duy trì dữ liệu được nhân bản từ các service để có thể dễ dàng query.
- Bằng cách định nghĩa một view database, định nghĩa một replica được thiết kế hỗ trợ việc query. Việc update data được duy trì bằng cách subscribe tới Domain events được publish bởi service sở hữu dữ liệu cần.
- Domain events là gì?
  - Một service cần publish event khi nó cập nhật dữ liệu, có thể dữ liệu đó để cập nhật vào table trong CQRS view

- Như đã nói ở trên có 2 patterns khác nhau giúp thực hiện việc truy vấn trên kiến trúc microservice đó là:
  - Api Composition pattern:
    - Đây là cách tiếp cận đơn giản và có thể sử dụng bất cứ khi nào có thể
    - Nó hoạt động bằng cách nhờ client đến của service sở hữu data, client này có trách nhiệm gọi đến service và tổng hợp dữ liệu.
  - CQRS pattern:
    - Pattern có sức mạnh hơn so với Api Composition pattern, tuy nhiên lại phức tạp hơn nhiều.
    - Tư tưởng chính của nó là duy trì nhiều table view database để support cho các query.

### Query sử dụng Api Composition pattern

> Thực hiện một query thực hiện truy vết dữ liệu từ một vài services bằng cách query mỗi service thông qua API của service và tổng hợp các kết quả truy vấn lại.

- Cấu trúc của Api Composition pattern

<p align="center">
    <img width="500" src="/detail/images/api-composition-structure.png" alt="">
</p>

- API composer:
  - Implement query bằng cách query data từ Provider Service và tổng hợp chúng
  - API Composer có thể là client ví dụ như web application nó cần dữ liệu để render page. Hoặc có thể là một service khác như API gateway, backend cho front-end.
- Provider Service: Service chứa dữ liệu
- Việc sử dụng pattern này để truy vấn dữ liệu còn phụ thuộc vào một số yếu tố ví dụ như sự phân vùng dữ liệu như thế nào, khả năng mà API của service cung cấp, khả năng của database mà service đó sử dụng.
- Ví dụ: Một `Provider service` có API hỗ trợ việc truy vết data, tuy nhiên việc kết hợp dữ liệu trong service thực hiện không hiệu quả, thực hiện in-memory với tập dataset lớn.

#### Implement query operation sử dụng API Composition pattern

<p align="center">
    <img width="500" src="/detail/images/implement-find-order.png" alt="">
</p>

- Api composer có một REST endpoint hoặc có thể sử dụng gRPC thay vì HTTP
- Các Service Provider cũng có một REST endpoint, Api Composer sẽ gọi đến các service này thông qua endpoint mà serive cung cấp. Các API này sẽ response về dữ liệu được kết hợp thông qua orderId mà Api composer truyền vào.
- Ai sẽ sử dụng Api composer?
  - Client ví dụ web application
  - API gateway
  - Chạy API comaposer trên một service độc lập
- Nhược điểm của Composition pattern
  - Chi phí cao do việc gọi nhiều service và truy vấn trên nhiều cơ sở dữ liệu.
  - Làm giảm tính khả dụng của hệ thống, ví dụ về findOrder(), nếu mỗi serive có tính khả dụng là 99.5% thì tính khả dụng của FindOrder Composer là (99.5)^5 = 97.5%. Có thể dùng kỹ thuật caching để tăng tính khả dụng của hệ thống, khi một Provider Service không sẵn sàng thì API Composer có thể trả về dữ liệu từ cache => dữ liệu này có thể đã cũ.
  - Thiếu tính nhất quán của dữ liệu, trong kiến trúc monolithic thì việc thực hiện nhiều truy vấn trên 1 cơ sở dữ liệu bằng cách thực hiện một transaction, transaction này đảm bảo thỏa tính ACID do đó dữ liệu luôn nhất quán cho view cho dù có nhiều query đi chăng nữa. Trong khi đó API Composition thực hiện nhiều câu truy vấn trên nhiều cơ sở dữ liệu khác nhau dẫn đến việc thiếu nhất quán.
- Có một số query yêu cầu API Composer thực hiện kết hợp in-memory với tập dữ liệu lớn thì API composition pattern không còn hiệu quả do đó cần một giải pháp như CQRS pattern.

### Query sử dụng CQRS pattern

#### Vấn đề truy vấn dữ liệu

- Giả sử có một operation findOrderHistory() thực hiện việc tìm tất cả các order của một customer, nó có một vài tham số như:
  - custormerId: để định danh khách hàng
  - pagination: số trang kết quả trả về
  - filter: lọc theo tiêu chí hay keyword nào đó.
- Câu query này trả về môt đối tượng OrderHistory có các thông tin về các đơn đặt hàng được sắp xếp theo thời gian của order.
- Truy vấn này trông có vẻ tương tự với findOrder(), điểm khác biệt duy nhất có lẽ là kết quả trả về của findOrder() là 1 kết quả duy nhất, trong khi đó findOrderHistory() có thể trả về nhiều order khác nhau.
- Vấn đề gặp phải ở đây đó là có một số service không có dữ liệu về trường cần filter, ở đây Order Service và Kitchen Service sẽ có thông tin về menu item của Order, trong khi đó Delivery Service và Accouting Service không có thông tin này, do đó không thể filter data thông qua keyword.
- Solution có vấn đề trên có thể là API Composer thực hiện join dữ liệu của Order Service và Kitchen Service với Delivery Service và Accouting Service

<p align="center">
    <img width="500" src="/detail/images/api-composition-problem.png" alt="">
</p>

- Nhược điểm là API Compose phải join một tập data lớn in-memory do đó cách giải quyết này không phù hợp.
- Solution thứ 2 là tiến hành truy vấn data ở Order Service và Kitchen Service sau đó lấy dữ liệu của 2 service còn lại thông qua orderId. Tuy nhiên điều này có thể không hiểu quả vì vượt quá network traffic.

#### Vấn đề truy vấn dữ liệu trên một service

- Như đã nói ở trên, vấn đề truy vấn dữ liệu trên nhiều serivce là một thách thức lớn, tuy nhiên việc query dữ liệu ở một service đôi khi cũng rất khó. Lý do là database của service sở hữu data không phù hợp cho việc query dữ liệu.
- Ví dụ: 
  - Có một câu truy vấn thực hiện việc tìm kiếm địa điểm của một nhà hàng, nếu Restaurant Service sử dụng một số database hỗ trợ lưu trữ Geospatial Data thì việc truy vấn dễ dàng, tuy nhiên nếu database của Service sử dụng loại database khác thì đây là một thách thức khác.
  - Có một giải pháp đó là duy trì một replica database lấy dữ liệu từ service database, điều này đồng nghĩa với việc đảm bảo consistent khi có update từ service database.

#### Tổng quan về CQRS

- Nhìn lại một số problem khi implement query trong một kiến trúc microservice:
  - Sử dụng API composition pattern để truy vấn dữ liệu trên nhiều service có chi phí cao, không hiểu quả khi tiến hành join in-memory.
  - Service lưu trữ dữ liệu trên một loại database nhất định, không phù hợp với hành động query.
  - Service chứa dữ liệu không nên là nơi cài đặt hành động query.

> Thực hiện query cần dữ liệu từ môt vài service bằng cách sử dụng các event để duy trì một read-only view, view này được replica từ các services.

#### CQRS tách Commands từ Queries

- Tách phần data model và những module sử dụng nó ra làm 2 phần: phía command và phía query
  - Command side có data model và các module hỗ trợ các hành động create, update, delete (CUD)
  - Query side có module và data model hỗ trợ việc query (R)

<p align="center">
    <img width="500" src="/detail/images/cqrs-model.png" alt="">
</p>

- Những service dựa trên nền tảng CQRS:
  - Command side:
    - Command-side model handles những thao tác CRUD và ánh xạ vào database của command model. Command side tiến hành push domain event ngay khi dữ liệu được thay đổi.
    - Events được push bằng cách sử dụng Event Sourcing hoặc framework có sẵn như Eventuate Tram.
  - Query-side:
    - Query handle những queries phức tạp.
    - Vì nó không implement một số nghiệp vụ nên query side sẽ ít phức tạp hơn so với command side.
    - Query side sử dụng một số loại database phù hợp với một số ngữ cảnh query.
    - Query side có handlers để handle sự kiện subscribe tới domain events và cập nhật một hoặc nhiều database.

#### CQRS và Service chỉ phục vụ cho việc query

- CQRS không chỉ áp dụng bên trong một service, mà ta cũng có thể sử dụng pattern này để định nghĩa một query service.
- Query service ở đây chỉ có API cho việc query, không có các command operations.
- Service này hiện thực query operation bằng cách query database được cập nhật dữ liệu nhờ vào subscribe tới events được publish bởi một hoặc nhiều service.
- Việc tách query-side service là một cách tốt để hiện thực một view, được xây dựng nhờ vào việc đăng ký sự kiện được publish bởi nhiều service khác. View này không còn là của một service nào cả,đây là một service độc lập.

<p align="center">
    <img width="500" src="/detail/images/query-service.png" alt="">
</p>

- Ví dụ ở đây ta có một Order History Service, Service này có cài đặt query findOrderHistory(). Service này đăng ký nhận các event được push từ một vài service, bao gồm các service có các thông tin cần thiết cho việc query Order History.

#### Lợi ích của CQRS

- CQRS có những điểm cộng sau:
  - Cho phép triển khai hiệu quả các truy vấn trong kiến trúc microservice.
    - Triển khai hiệu quả các truy vấn cần truy vết dữ liệu ở nhiều service khác nhau.
    - Hiệu quả hơn so với khi sử dụng API Composition.
  - Cho phép triển khai hiệu quả các truy vấn đa dạng
    - Một số NoSQL databases khả năng query bị giới hạn.
    - Ngay cả khi database có phần mở rộng hỗ trợ một số loại query, thì việc sử dụng database chuyên dụng cho loại dữ liệu đó thì luôn hiệu quả hơn.
    - CQRS pattern tránh việc giới hạn trong một datastore bằng cách định nghĩa nhiều views, mỗi view sẽ hiệu quả cho từng loại truy vấn đặc biệt.
  - Có thể truy vấn trong những ứng dụng dựa trên nền tảng event sourcing.
    - CQRS khắc phục một số nhược điểm của event-sourcing, event store chỉ hỗ trợ những truy vấn dựa trên khóa chính.
    - CQRS giải quyết vấn đề này bằng các định nghĩa một hay nhiều tổ hợp view, được cập nhật bằng cách đăng ký là luồng sự kiện được publish bởi các tổ hợp dựa trên event-sourcing.
  - Cải thiện việc tách biệt các concerns.
    - Một trong những lợi ích của CQRS đó là tách biệt các concerns.
    - Domain model và data model không handle cả command lẫn queries, CQRS pattern chia code modules và lược đồ database cho command và query side của một service. Bằng cách chia ra command side và query side trở nên dễ dàng để bảo trì, phát triển.
    - Ngoài ra service triển khai query sẽ khác so với service sở hữu dữ liệu.

#### Nhược điểm của CQRS

- Mặc dù có những ưu điểm vượt trồi, những CQRS cũng có những nhược điểm gây khó khăn như:
  - Làm cho kiến trúc trở nên phức tạp hơn
    - Lập trình viên phải viết query-side services để mà update và query view.
    - Khi application có thể sử dụng những loại database khác nhau, nó lại làm tăng thêm phần phức tạp cho các oeprations.
  - Đối mặt với Replication Lag
    - Hiện tượng lag giữa command-side và query-side. Ta luôn mong đợi rằng khi command side publish một sự kiện và khi sự kiện được query side xử lý xong thì view sẽ được update. Vấn đề là làm sao tránh được tình trạng inconsistency khi client app update.

## Event Sourcing

### Problem

- Môt service cần tự động cập nhật database và push message/events. Nếu sử dụng Saga pattern thì mỗi bước thực hiện trong saga phải tiến hành cập nhật database cũng như push message/events. Một giải pháp thay thể Sage đó là sử dụng Domain Event (xem thêm ở [đây](https://microservices.io/patterns/data/domain-event.html)), để triển khai CQRS.
- Vấn đề ở đây là làm thế nào để tự động cập nhật database và publish messages/events.

### Solution

- Để giải quyết vấn đề này, ta sử dụng event sourcing, event sourcing lưu trữ trạng thái của một business entity thành một chuỗi các sự kiện thay đổi của state. Một khi trạng thái của một business entity thay đổi, một sự kiện mới được append vào danh sách sự kiện. Vì các sự kiện được lưu đơn lẻ do đó đảm bảo atomic, application có thể dựng lại trạng thái hiện tại bằng cách chạy lại các event trong event lists.

#### Event Sourcing Persists aggregation using events

- Event sourcing là một kỹ thuật event-centric để triển khai business logic và lưu trữ tập hợp dữ liệu. Tập trạng thái được lưu trữ ở database dưới dạng một chuỗi các event.
- Các cách tiếp cận truyền thống, một lớp được map vào trong một bảng, trong đó field của lớp là các cột, và value tương ứng với cột là dòng. Trong khi đó event sourcing xây dựng trên tư tưởng của domain event. Lưu trữ trạng thái của một đối tượng như là một chuỗi các sự kiện trong database, hay còn gọi là event store.

<p align="center">
    <img width="500" src="/detail/images/event-sourcing-example.png" alt="">
</p>

- Khi ứng dụng tạo hoặc cập nhật một state của một đối tượng, nó chèn một sự kiện được tạo ra bởi đối tượng đó vào Events table. Ứng dụng xây dựng lại đối tượng bằng cách truy vết các sự kiện và xây dựng lại.

#### Event represent state changes

- Domain events như là một cơ chế để thông báo cho các subscriber khi có thay đổi. Mỗi event có thể kèm theo một số dữ liệu chẳng hạn như aggregate ID, hoặc một số data hữu ích khác.
- Ví dụ: Order Service có thể push một event là OrderCreated, OrderCreated có chứa orderId, khi event chứa Id thì consumer không cần phải fetch vào Order Service để lấy thông tin.
- Event cần phải có data của đối tượng cần chuyển đổi, ví dụ như thay đổi trạng thái đặt hàng, thì trong sự kiện phải có giá trị của trạng thái.

<p align="center">
    <img width="500" src="/detail/images/condition-event.png" alt="">
</p>

#### Aggregate methods are all about events

- Khi business login yêu cầu cập nhật một đối tượng bằng cách gọi thông qua command method, trong các ứng dụng thông thường thì ta sẽ kiểm tra đối số truyền vào, sau đó cập nhật các trường của đối tượng. Command method của các ứng dụng dựa trên event sourcing sẽ tạo ra các event, kết quả của mỗi lần thực hiện command method là một dãy các sự kiện nó thể hiện sự thay đổi trạng thái và phải được thực hiện. Các event được lưu trữ vào event store và được sử dụng để thay đổi state của đối tượng.

<p align="center">
    <img width="500" src="/detail/images/method-event-sourcing.png" alt="">
</p>

- Thực hiện một command chỉ tạo ra các event mà không thay đổi trạng thái của đối tượng, đối tượng được cập nhật trạng thái bằng cách áp dụng event trong event store.
- Event Sourcing phân chia command method thành hai hoặc nhiều method, method thứ nhất lấy các tham số của request và xác định trạng thái nào cần thay đổi để thực hiện. Tuy nhiên method này không thay đổi trạng thái của đối tượng, nó trả về danh sách các sự kiện làm thay đổi trạng thái, nếu command không thể thực hiện thì nó sẽ throw một exception.
- Method khác nhận đối số là Event và cập nhật đối tượng, method luôn thực hiện thành công bởi vì event này đã xảy ra trước đó.

#### Handling concurrent updates using optimistic locking

- Optimistic locking được sử dụng để ngăn chặn trường hợp có đồng thời 2 command cùng thay đổi trạng thái của một đối tượng.

#### Event sourcing and publishing events

