Example: [https://jira.zalopay.vn/browse/PMTBT-4311](https://jira.zalopay.vn/browse/PMTBT-4311)

# Root cause:

HMAC Input:

```Java
String[] hmacInput = new String[]{
        appID + "",
        appTransID,
        appUser,
        amount + "",
        appTime + "",
        embedData,
        item,
};
String mac = HMACUtils.hmacHexStringEncode(algorithm, hashKey, String.join("|", hmacInput));
```

  

Trong `hmacInput` có chứa kí tự unicode lại nên khi tính hmac ở Golang project sẽ cho ra kết quả khác hmac khi tính ở Java project

  

# How to resolve?

- [ ] Kiểm tra kí tự unicode ở đâu: thường sẽ nằm ở 2 thông tin `embedData` hoặc `item`
- [ ] Nên sử dụng chrome thay vì safari
- [ ] Sử dụng [https://text-compare.com/](https://text-compare.com/) để compare 2 đoạn (tự nhập và user nhập)
- [ ] Sử dụng [https://r12a.github.io/app-conversion/](https://r12a.github.io/app-conversion/) để đếm số lượng kí tự của đoạn text

# Solution

```Java

@Test
public void testNormalize(){
    String source = "PHAN VIỆT HƯNG";
    String des = "PHAN VIỆT HƯNG";

    System.out.println(Normalizer.normalize(source, Normalizer.Form.NFC));
		// PHAN VIỆT HƯNG
    System.out.println(Normalizer.normalize(source, Normalizer.Form.NFD));
		// PHAN VIỆT HƯNG
    System.out.println(Normalizer.normalize(source, Normalizer.Form.NFKC));
		// PHAN VIỆT HƯNG
    System.out.println(Normalizer.normalize(source, Normalizer.Form.NFKD));
		// PHAN VIỆT HƯNG
}
```