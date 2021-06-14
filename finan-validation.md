## Backend Issue

### Summary
Bữa nay tạm ngưng Hackathon ngồi audit Backend API, phát hiện ra lỗi validation.
- Lỗi này cho phép bạn đặt đơn hàng với giá chỉ vài  đồng

### Detail:
- Bắt đầu với API tạo đơn hàng trị giá 204900.

<details>
<summary>Valid request</summary>

```json
POST /create-order
{
    "business_id": "69417842-5c1d-417e-9e58-01bfb38a38f1",
    "promotion_code": null,
    "ordered_grand_total": 204900,
    "promotion_discount": 0,
    "delivery_fee":0,
    "grand_total": 204900,
    "state": "waiting_confirm",
    "payment_method": "COD",
    "note": "Nhận hàng ở đầu hẻm.", 
    "delivery_method":"seller_delivery",
    "buyer_info": {
        "phone_number": "0899329946",
        "name": "Nguyễn Văn Minh",
        "address": "Central park 3"
    },
    "list_order_item": [
        {
            "product_id":"d18e006e-c9be-440d-946e-6bb0bad00c78",
            "product_name":"Bánh xèo",
            "product_selling_price": 4900,
            "product_normal_price": 5000,
            "quantity":1,
            "product_images": [
                "https://d3hr4eej8cfgwy.cloudfront.net/centralweb/4d10b423-0350-41c7-9b5f-d4b6bd2fde87/image/1623145905416.png"
            ],
            "note":"bánh nhiều"
        },
         {
            "product_id":"16f6e859-e20a-4013-906c-294e7efd0c6f",
            "product_name":"Cơm gia đình",
            "product_normal_price":80000,
            "product_selling_price":0,
            "product_images": [
                "https://d3hr4eej8cfgwy.cloudfront.net/centralweb/4d10b423-0350-41c7-9b5f-d4b6bd2fde87/image/1623144732667.png"
            ],
            "quantity":2,
            "note":"Không cay"
        },
         {
            "product_id":"eedfe0a3-b40d-4d5b-b3f8-200234381739",
            "product_name":"Mì tôm",
            "product_normal_price":40000,
            "product_selling_price":0,
            "product_images": [
                "https://d3hr4eej8cfgwy.cloudfront.net/centralweb/4d10b423-0350-41c7-9b5f-d4b6bd2fde87/image/1623145312183.png"
            ],
            "quantity":1,
            "note":"Mì chín kỹ"
        
        }

    ]
}
```
</details>

- Bây giờ cập nhật selling price cho mỗi product thành 1, vì đơn hàng có 4 items => cập nhật ordered_grand_total =4 & grand_total = 4. Submit, hệ thống ghi nhận đơn hàng 4 đồng.
Thực tế mình có thể làm đơn hàng có giá hợp lý hơn, seller sẽ không nhận ra và xử lý đơn hàng.

<details>
<summary>Injection request</summary>

```json
{
    "business_id": "69417842-5c1d-417e-9e58-01bfb38a38f1",
    "promotion_code": null,
    "ordered_grand_total": 4,
    "promotion_discount": 0,
    "delivery_fee":0,
    "grand_total": 4,
    "state": "waiting_confirm",
    "payment_method": "COD",
    "note": "Nhận hàng ở đầu hẻm.", 
    "delivery_method":"seller_delivery",
    "buyer_info": {
        "phone_number": "0899329946",
        "name": "Nguyễn Văn Minh",
        "address": "Central park 3"
    },
    "list_order_item": [
        {
            "product_id":"d18e006e-c9be-440d-946e-6bb0bad00c78",
            "product_name":"Bánh xèo",
            "product_selling_price": 1,
            "product_normal_price": 5000,
            "quantity":1,
            "product_images": [
                "https://d3hr4eej8cfgwy.cloudfront.net/centralweb/4d10b423-0350-41c7-9b5f-d4b6bd2fde87/image/1623145905416.png"
            ],
            "note":"bánh nhiều"
        },
         {
            "product_id":"16f6e859-e20a-4013-906c-294e7efd0c6f",
            "product_name":"Cơm gia đình",
            "product_normal_price":80000,
            "product_selling_price":1,
            "product_images": [
                "https://d3hr4eej8cfgwy.cloudfront.net/centralweb/4d10b423-0350-41c7-9b5f-d4b6bd2fde87/image/1623144732667.png"
            ],
            "quantity":2,
            "note":"Không cay"
        },
         {
            "product_id":"eedfe0a3-b40d-4d5b-b3f8-200234381739",
            "product_name":"Mì tôm",
            "product_normal_price":40000,
            "product_selling_price":1,
            "product_images": [
                "https://d3hr4eej8cfgwy.cloudfront.net/centralweb/4d10b423-0350-41c7-9b5f-d4b6bd2fde87/image/1623145312183.png"
            ],
            "quantity":1,
            "note":"Mì chín kỹ"
        
        }

    ]
}
```
</details>

### Never trust your users
Anh phát hiện ra bug này vì:
- Anh thấy có quá nhiều data gửi lên từ client (users site). Ví dụ: ảnh, giá bán, tên sản phẩm là không cần thiết.
- Kể cả có gửi lên thì không sao, nhưng không nên trust dữ liệu này. Thay vào đó mình vẫn cần query lại trên Backend để lấy thông tin sản phẩm.
- Càng nhiều dữ liệu gửi lên từ client càng dễ mắc lỗi trong xử lý validation: nếu mình không gửi giá sản phẩm nên, chắc chắn Backend sẽ phải lẫy thông tin sản phẩm.

