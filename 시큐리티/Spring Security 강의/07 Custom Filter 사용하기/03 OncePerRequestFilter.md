# OncePerRequestFilter

- 단 한 번의 실행: 이 필터는 동일한 요청에 대해 여러 번 실행되지 않도록 보장됩니다.   
- doFilterInterval(): 기존 Filter 인터페이스를 상속하여 만든 CustomFilter는 doFilter() 메서드에 비지니스 로직을 작성했다.   
하지만 OncePerRequestFilters는 이미 doFilter가 작성되어있다. 이럴 경우 어디에 비지니스 로직을 작성해야할까?   
바로 **doFilterInterval() 메소드 안에 작성하면 된다!**

```java
public abstract class OncePerRequestFilter extends GenericFilterBean {
    public static final String ALREADY_FILTERED_SUFFIX = ".FILTERED";

    public OncePerRequestFilter() {
    }

    public final void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        if (request instanceof HttpServletRequest httpRequest) {
            if (response instanceof HttpServletResponse httpResponse) {
                String alreadyFilteredAttributeName = this.getAlreadyFilteredAttributeName();
                boolean hasAlreadyFilteredAttribute = request.getAttribute(alreadyFilteredAttributeName) != null;
                if (!this.skipDispatch(httpRequest) && !this.shouldNotFilter(httpRequest)) {
                    if (hasAlreadyFilteredAttribute) {
                        if (DispatcherType.ERROR.equals(request.getDispatcherType())) {
                            this.doFilterNestedErrorDispatch(httpRequest, httpResponse, filterChain);
                            return;
                        }

                        filterChain.doFilter(request, response);
                    } else {
                        request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);

                        try {
                            this.doFilterInternal(httpRequest, httpResponse, filterChain);
                        } finally {
                            request.removeAttribute(alreadyFilteredAttributeName);
                        }
                    }
                } else {
                    filterChain.doFilter(request, response);
                }

                return;
            }
        }

        throw new ServletException("OncePerRequestFilter only supports HTTP requests");
    }

    private boolean skipDispatch(HttpServletRequest request) {
        if (this.isAsyncDispatch(request) && this.shouldNotFilterAsyncDispatch()) {
            return true;
        } else {
            return request.getAttribute("jakarta.servlet.error.request_uri") != null && this.shouldNotFilterErrorDispatch();
        }
    }

    protected boolean isAsyncDispatch(HttpServletRequest request) {
        return DispatcherType.ASYNC.equals(request.getDispatcherType());
    }

    protected boolean isAsyncStarted(HttpServletRequest request) {
        return WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted();
    }

    protected String getAlreadyFilteredAttributeName() {
        String name = this.getFilterName();
        if (name == null) {
            name = this.getClass().getName();
        }

        return name + ".FILTERED";
    }

    protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
        return false;
    }

    protected boolean shouldNotFilterAsyncDispatch() {
        return true;
    }

    protected boolean shouldNotFilterErrorDispatch() {
        return true;
    }

    protected abstract void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException;

    protected void doFilterNestedErrorDispatch(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        filterChain.doFilter(request, response);
    }
}

```