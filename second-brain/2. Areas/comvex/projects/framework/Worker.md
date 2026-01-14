
Config
Routing Key

Queue name
so we will have many queue name and each will have routing key and each queue name 
Messy because handle will register by routing key but the queue name will only using for message
like I have two queue "foo"and "bar" but I only have one routing key for ex "routing", now if the message send to `foo` or `bar` I still can get and process it 


What the fuck is processor

processor wrap rabbitmq consumer 

So that processor is consumer  and the server is just a list of consumer .


Đang start 2 consumer và mỗi consumer sẽ có một msg_handler xử lý message.
Vấn đề là có cần 2 msg_handler hay không hay chỉ cần một consumer và message handler????
Nếu như một lỗi và một ko lỗi thì sao?????
Lúc đó thì làm gì????

Case ntn, fetch đc message từ rabbitMQ, 
Xử lý


Channel Cancel will stop
Tại vì có preload nên là sẽ đợi lượng preload xử lý hết sau đó mới stop channel.

Thứ tự sẽ là cancel channel, waitAllMessage done, close channel, close connection.

T consume queue này, thằng kia consume queue kia

Vì mình setup rất nhiều consumer nên lão muốn là khi mà setup xong hết mới bắt đầu consumer. Lỡ như một consumer fail thì mấy thằng khác không cần consumer


Chỉ có 1 channel nhưng mà tạo tới 2 consumer


1 Processor tương ứng 1 queue name???? really????


Khi Stop thì sao

cancel channel, waitAllMessage done, close channel, close connection.

start app consumer 
Setup all processor then start all consumer
Theo queue name,
Lỡ duplicate queue name thì sao?


If one processor - stop using the cancelFunc in argument to stop the whole server No, so what we did here ????

ProccessorMap using routing key so that if the queue don't have that routing key still can handler by other handler







<<<<<<< HEAD

=======
When there is an error in consumer, only one consummer effect.
>>>>>>> 41ec96b (vault backup: 2026-01-14 09:58:32)
