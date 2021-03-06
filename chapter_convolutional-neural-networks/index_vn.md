<!-- ===================== Bắt đầu dịch  ==================== -->
<!-- ========================================= REVISE BẮT ĐẦU =================================== -->

<!--
# Convolutional Neural Networks
-->

# Mạng nơ-ron Tích chập
:label:`chap_cnn`

<!--
In earlier chapters, we came up against image data, for which each example consists of a 2D grid of pixels.
Depending on whether we're handling black-and-white or color images, each pixel location might be associated with either *one* or *multiple* numerical values, respectively.
Until now, our way of dealing with this rich structure was deeply unsatisfying.
We simply discarded each image's spatial structure by flattening them into 1D vectors, feeding them through a (fully connected) MLP.
Because these networks invariant to the order of the features we could get similar results regardless of whether 
we preserve an order corresponding ot the spatial structure of the pixels or if we permute the columns of our design matrix before fitting the MLP's parameters.
Preferably, we would leverage our prior knowledge that nearby pixels are typically related to each other, to build efficient models for learning from image data.
-->

Tronng các chương đầu tiên này, chúng ta đã tiếp cận với dữ liệu ảnh mà mỗi mẫu bao gồm một mảng điểm ảnh 2 chiều.
Tùy vào loại ảnh là trắng đen hay ảnh màu mà ta cần xử lý *một* hay *nhiều* giá trị số học tương ứng tại mỗi vị trí điểm ảnh . 
Cho đến lúc này, cách thức chúng ta xử lý dữ liệu với cấu trúc phong phú này vẫn chưa thực sự thoả đáng.
Ta chỉ đơn thuần loại bỏ cấu trúc không gian của mỗi bức ảnh bằng cách chuyển chúng thành các vector 1 chiều và truyền chúng qua một mạng MLP (kết nối đầy đủ).
Bởi vì các mạng này là bất biến với thứ tự của các đặc trưng, ta sẽ nhận được cùng một kết quả bất chấp việc chúng ta có giữ lại thứ tự cấu trúc không gian của các điểm ảnh hay hoán vị các cột của ma trận đặc trưng trước khi khớp các tham số của mạng MLP. 
Nếu có thể dược, ta nên tận dụng điều đã biết là các điểm ảnh kề cận thường có liên hệ lẫn nhau, để xây dựng những mô hình hiệu quả hơn cho việc học từ dữ liệu ảnh.

<!--
This chapter introduces convolutional neural networks (CNNs), a powerful family of neural networks that were designed for precisely this purpose.
CNN-based architectures are now ubiquitous in the field of computer vision, and have become so dominant that hardly anyone 
today would develop a commercial application or enter a competition related to image recognition, object detection, or semantic segmentation, without building off of this approach.
-->

Chương này giới thiệu về các mạng nơ-ron tích chập (_Convolutional neural network - CNN_), một họ các mạng nơ-ron đầy sức mạnh được thiết kế với chính xác cho mục đích trên. 
Các kiến trúc dựa trên CNN hiện nay xuất hiện trong mọi ngóc ngách của lĩnh vực thị giác máy tính, và đã trở thành chủ đạo mà hiếm có ai ngày nay khi phát triển các ứng dụng thương mại hay tham gia một cuộc thi nào đó liên quan tới nhận dạng ảnh, phát hiện đối tượng, hay phân vùng theo ngữ cảnh mà không xây nền móng bằng phương pháp này.

<!--
Modern *ConvNets*, as they are called colloquially owe their design to inspirations from biology, group theory, and a healthy dose of experimental tinkering.
In addition to their sample efficiency in achieving accurate models, convolutional neural networks tend to be computationally efficient, 
both because they require fewer parameters than dense architectures and because convolutions are easy to parallelize across GPU cores.
Consequently, practitioners often apply CNNs whenever possible, and increasingly they have emerged as credible competitors even on tasks with 1D sequence structure, 
such as audio, text, and time series analysis, where recurrent neural networks are conventionally used.
Some clever adaptations of CNNs have also brought them to bear on graph-structured data and in recommender systems.
-->

Theo cách gọi thông dụng ngày nay, thiết kế của mạng *ConvNets* đã vay mượn rất nhiều ý tưởng từ ngành sinh học, lý thuyết nhóm và lượng không nhỏ những thí nghiệm chắp vá.
Bênh cạnh hiệu năng cao trên số lượng mẫu cần thiết để đạt được đủ độ chính xác, các mạng nơ-ron tích chập còn rất hiệu quả về mặt tính toán, bởi mạng tích chập đòi hỏi lượng tham số ít hơn và dễ thực thi song song trên nhiều GPU hơn các kiến trúc mạng dày đặc. 
Do đó, những người hành nghề thường áp dụng các mạng CNN bất cứ khi nào có thể, và chúng đã nhanh chóng trở thành một công cụ quan trọng đáng tin cậy thậm chí với các tác vụ liên quan tới cấu trúc tuần tự một chiều,
như là xử lý âm thanh, văn bản, và phân tích dữ liệu chuỗi thời gian (_time series analysis_), mà ở đó các mạng nơ-rơn truy hồi vốn thường được sử dụng. 
Với một số điều chỉnh khôn khéo còn cho phép sử dụng các mạng CNN với dữ liệu có cấu trúc đồ thị và trong các hệ thống đề xuất.

<!--
First, we will walk through the basic operations that comprise the backbone of all convolutional networks.
These include the convolutional layers themselves, nitty-gritty details including padding and stride, 
the pooling layers used to aggregate information across adjacent spatial regions, the use of multiple *channels* (also called *filters*) at each layer, 
and a careful discussion of the structure of modern architectures.
We will conclude the chapter with a full working example of LeNet, the first convolutional network successfully deployed, long before the rise of modern deep learning.
In the next chapter, we will dive into full implementations of some popular and 
comparatively recent CNN architectures whose designs representat most of the techniques commonly used by modern practitioners.
-->

Trước hết, chúng ta sẽ đi qua các phép toán cơ bản tạo nên bộ khung sườn của tất cả các mạng nơ-ron tích chập.
Chúng bao gồm các chính tầng tích chập, các chi tiết cơ bản quan trọng như đệm và sải bước, các tầng gộp dùng để kết hợp thông tin qua các vùng không gian kề nhau, việc sử dụng đa kênh (cũng được gọi là *các bộ lọc*) ở mỗi tầng và một cuộc thảo luận cẩn thận về cấu trúc của các mạng hiện đại. 
Chúng ta sẽ kết thúc cho chương này với một ví dụ hoàn toàn hoạt động của mạng LeNet, mạng tích chập đầu tiên đã triển khai thành công và tồn tại nhiều năm trước khi có sự trỗi dậy của kỹ thuật học sâu hiện đại. 
Ở chương kế tiếp, chúng ta sẽ đắm mình vào việc xây dựng hoàn chỉnh một số kiến trúc CNN tương đối gần đây và khá phổ biến mà những thiết kế này thể hiện hầu hết các kỹ thuật được sử dụng bởi những người làm hành nghề hiện nay. 

```toc
:maxdepth: 2

why-conv_vn
conv-layer_vn
padding-and-strides_vn
channels_vn
pooling_vn
lenet_vn
```

<!-- ===================== Kết thúc dịch  ==================== -->
<!-- ========================================= REVISE KẾT THÚC ===================================-->

## Những người thực hiện
Bản dịch trong trang này được thực hiện bởi:
<!--
Tác giả của mỗi Pull Request điền tên mình và tên những người review mà bạn thấy
hữu ích vào từng phần tương ứng. Mỗi dòng một tên, bắt đầu bằng dấu `*`.

Lưu ý:
* Nếu reviewer không cung cấp tên, bạn có thể dùng tên tài khoản GitHub của họ
với dấu `@` ở đầu. Ví dụ: @aivivn.

* Tên đầy đủ của các reviewer có thể được tìm thấy tại https://github.com/aivivn/d2l-vn/blob/master/docs/contributors_info.md
-->

* Đoàn Võ Duy Thanh
* Nguyễn Mai Hoàng Long
* Lê Khắc Hồng Phúc
