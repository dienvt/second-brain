package dto

  // InvoiceResponse represents a Vietnamese VAT invoice response.
  type InvoiceResponse struct {
        Items         []InvoiceLineItem `json:"items"`
        Subtotal      float64           `json:"subtotal"`       // Cộng tiền hàng
        VATRate       float64           `json:"vatRate"`        // Thuế suất GTGT (e.g., 0.08 for 8%)
        VATAmount     float64           `json:"vatAmount"`      // Tiền thuế GTGT
        TotalPayment  float64           `json:"totalPayment"`   // Tổng cộng tiền thanh toán
        AmountInWords string            `json:"amountInWords"`  // Số tiền bằng chữ
        Currency      string            `json:"currency"`       // e.g., "VND"
  }

  // InvoiceLineItem represents a single row in the invoice table.
  type InvoiceLineItem struct {
        No         int     `json:"no"`         // STT
        Name       string  `json:"name"`       // Tên hàng hóa, dịch vụ
        Unit       string  `json:"unit"`       // Đơn vị tính
        Quantity   float64 `json:"quantity"`   // Số lượng
        UnitPrice  float64 `json:"unitPrice"`  // Đơn giá
        Amount     float64 `json:"amount"`     // Thành tiền = quantity * unitPrice
  }

  Populated with the image data:

  InvoiceResponse{
      Items: []InvoiceLineItem{
          {
              No:        1,
              Name:      `Vé sự kiện "VIETNAM TRIFACTOR 2026" - TRIFACTOR ENDURO (SOLO) - TIER 1 (Swim 2250m – Bike 80km – Run 20km)`,
              Unit:      "Vé",
              Quantity:  1,
              UnitPrice: 3_240_741,
              Amount:    3_240_741,
          },
      },
      Subtotal:      2_916_667,
      VATRate:       0.08,
      VATAmount:     233_333,
      TotalPayment:  3_150_000,
      AmountInWords: "Ba triệu một trăm năm mươi nghìn đồng",
      Currency:      "VND",
  }

  Notes on design choices:
  - float64 for money is simple but imprecise — if this handles real billing, switch to decimal.Decimal (github.com/shopspring/decimal) or integer minor units. VND has no subunit so int64
  is viable.
  - VATRate as decimal (0.08) rather than percent (8) — keeps arithmetic direct; format for display at the edge.
  - Amount kept on the line item rather than computed — matches what the invoice document asserts (source of truth), avoids floating-point drift on echo-back.
  - Quantity as float64 covers fractional units (hours, kg); use int if your domain is strictly whole units.
