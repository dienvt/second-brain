Giới thiệu về DDD và cách áp dụng
Đa số thông tin là từ cuốn sách gốc của Chris Evan. Ông này chỉ viết lại tóm tắt cách hiểu của ổng

## Uniquious Language
Chương đầu nói về việc Dev và BA phải nói chung một ngôn ngữ. Như vậy thì dev mới hiểu hết được về Domain (bằng cách đặt câu hỏi để làm rõ vấn đề).
Trong cuốn sách này cũng ko có đề xuất về ngôn ngữ riêng. Nó chỉ bàn luận về khó khăn khi Dev nói ngôn ngữ của dev (class, object, ...) không làm rõ được vấn đề
Ngoài ra thì cũng có nói về khó khăn khi sử dụng UML, đối với dự án lớn thì việc maintain UML rất khó khăn và nên tìm một giải pháp khác
## Model Driven Design
Giới thiệu về kiến trúc phân lớp

#### Entity
- Phải có định danh
- Chứa thuộc tính, tham chiếu có thể thay đổi.
- Là thành phần quan trọng phải được xác định từ đầu
- Mutable
#### Value Object:
- immutable

#### Aggregate
- Chứa một **Aggregate Root** là một **Entity** và những entity liên quan
- ex; Order và Order Lines (items)
	- Aggreate (order)
		- Aggreate Root: Order
		- Related Entity: Order Lines
#### Factory


#### Repository


### Preserving Model Integrity
Giowois
Giới thiệu các technque để mà đảo bảo tính toàn vẹn của model

**Bounded context**
**Context Map**
Cách technque được giới thiệu theo những mức độ phối hợp giữa hai team (hai bounded context):
* Dính với nhau (High coupling):
	* Shared kernel
- Nếu như 2 team làm việc closely
	- Supplỉe/ comsuemr
- Nếu như 1 téam ko thể fullfill team kia
	- Conformist
	- Supplier will provide they model
	- Consumer has to convert the Supplier's model in to the Consumer's model
- Giao tiếp với External / Legacy system
	- Anti-Corruption Layer
- External giao tiếp với mình
	- Open Host Service
	- Define API constraint
	- Thường đi đôi với Distillation
		- Forcus vào core logic



“names enter

UBIQUITOUS LANGUAGE

keep model unified by -

CONTINUOUS INTEGRATION

BOUNDED CONTEXT

SHARED KERNEL

assess/overview relationships with

CUSTOMER SUPPLIER

TEAMs

CONTEXT MAP

overlap allied contexts through relate allied contexts as

overlap unilaterally as

support multiple clients through

free teams to go

translate and insulate unilaterally with

OPEN HOST

SERVICE

CONFORMIST

formalize as

ANTICORRUPTION

LAYER

SEPARATE

WAYS

PUBLISHED LANGUAGE”

  

Excerpt From

DDD Quickly

This material may be protected by copyright.
