### HshcFrameWorkFilter

```java
private void selfDoFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        if (request instanceof HttpServletRequest && response instanceof HttpServletResponse) {
            HttpServletRequest req = (HttpServletRequest)request;
            HttpServletResponse res = (HttpServletResponse)response;
            if (!HSHCSysHandleUtil.invoke(req, res, chain)) {
                String url = this.getCollectUrl(req.getRequestURI());
                SectionNode node = SectionMgr.getCon().putInvokeNode(url);
                long time = System.currentTimeMillis();
                boolean su = false;

                try {
                    HSHCAppContext.setControllerContextId(url);
                    ServletContext application = request.getServletContext();
                    setUrlsToApplaction(application);
                    HttpServletRequest realInvokeReq = req;
                    if (SessionUtil.isOpen()) {
                        String id = req.getRequestedSessionId();
                        if (id == null) {
                            id = req.getSession(true).getId();
                        }

                        HshcHttpRequestWrapper wrapper = new HshcHttpRequestWrapper(req, id);
                        realInvokeReq = wrapper;

                        try {
                            SessionUtil.flushSession(wrapper);
                        } catch (Exception var19) {
                            logger.error("flush session timeout error!", var19);
                        }
                    }

                    HttpContext.setReqAndRes((HttpServletRequest)realInvokeReq, res);
                    chain.doFilter((ServletRequest)realInvokeReq, response);
                    su = true;
                } finally {
                    node.remove();
                    PerformLogCollector.getInstance().collectCtr(url, su, time, System.currentTimeMillis());
                    HSHCAppContext.removeContext();
                    HttpContext.remove();
                }
            }
        } else {
            chain.doFilter(request, response);
        }

    }
    ```
