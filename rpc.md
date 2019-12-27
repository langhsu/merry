### RemoteServiceInvokeUtil

```java
public static Object doHttpInvoke(RSClientParam clientParam) throws Exception {
	String url = getUrl(clientParam);
	
	// 封装请求体(请求参数、链路信息)
	RemoteServiceParam param = RemoteClientUtil.getByClientParam(clientParam);

	try {
		// 执行网络请求
		V2<Integer, byte[]> v2 = doGet(url, GqSeUtil.serialize(param), clientParam.getConnTimeout(), clientParam.getReadTimeout());
		byte[] bytes = (byte[])v2.getV2();
		if ((Integer)v2.getV1() == 200) {
			try {
				Object obj = GqSeUtil.deserialize(bytes);
				return obj;
			} catch (Exception var6) {
				logger.error("数据转换错误 serviceId=" + clientParam.getServiceId() + ",bytes=" + Arrays.toString(bytes), var6);
				throw var6;
			}
		} else {
			String msg = null;
			if (v2.getV2() == null) {
				msg = new String((byte[])v2.getV2());
			}
			throw new RSClientException(clientParam, "服务端错误code=" + v2.getV1() + ",msg=" + msg);
		}
	} catch (Exception var7) {
		throw new Exception("远程服务错误,url=" + url + ",msg=" + var7.getMessage(), var7);
	}
}
```
