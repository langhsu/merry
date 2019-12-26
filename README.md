### HshcFrameWorkFilter

```java
private void selfDoFilter(ServletRequest request, ServletResponse response, FilterChain chain){
	if (request instanceof HttpServletRequest && response instanceof HttpServletResponse) {
		// ...
		if (!HSHCSysHandleUtil.invoke(req, res, chain)) {
			// ...
			long time = System.currentTimeMillis();
			boolean su = false;

			try {
				// 设置上下文, 生成链路sessionId
				HSHCAppContext.setControllerContextId(url);
				// ...
				HttpContext.setReqAndRes((HttpServletRequest)realInvokeReq, res);
				chain.doFilter((ServletRequest)realInvokeReq, response);
				su = true;
			} finally {
				node.remove();
				// 发送性能日志
				PerformLogCollector.getInstance().collectCtr(url, su, time, System.currentTimeMillis());
				// 清理上下文信息
				HSHCAppContext.removeContext();
				HttpContext.remove();
			}
		}
	} else {
		chain.doFilter(request, response);
	}
}
```
