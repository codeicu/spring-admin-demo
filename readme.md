# 测试spring-admin

- 问题1. client url是如何确定的?

由de.codecentric.boot.admin.client.registration.DefaultApplicationFactory决定IP地址,

关键代码如下:
```

    protected InetAddress getLocalHost() {
        try {
            return InetAddress.getLocalHost();
        }
        catch (UnknownHostException ex) {
            throw new IllegalArgumentException(ex.getMessage(), ex);
        }
    }
	
    protected String getHost(InetAddress address) {
        return this.instance.isPreferIp() ? address.getHostAddress() : address.getCanonicalHostName();
    }
```

完整代码如下
```
	@Override
	public Application createApplication() {
		return Application.create(getName()).healthUrl(getHealthUrl()).managementUrl(getManagementUrl())
				.serviceUrl(getServiceUrl()).metadata(getMetadata()).build();
	}

	protected String getName() {
		return this.instance.getName();
	}

	protected String getServiceUrl() {
		if (this.instance.getServiceUrl() != null) {
			return this.instance.getServiceUrl();
		}

		return UriComponentsBuilder.fromUriString(getServiceBaseUrl()).path(getServicePath()).toUriString();
	}

	protected String getServiceBaseUrl() {
		String baseUrl = this.instance.getServiceBaseUrl();

		if (StringUtils.hasText(baseUrl)) {
			return baseUrl;
		}

		return UriComponentsBuilder.newInstance().scheme(getScheme(this.server.getSsl())).host(getServiceHost())
				.port(getLocalServerPort()).toUriString();
	}

	protected String getServicePath() {
		String path = this.instance.getServicePath();

		if (StringUtils.hasText(path)) {
			return path;
		}

		return "/";
	}

	protected String getManagementUrl() {
		if (this.instance.getManagementUrl() != null) {
			return this.instance.getManagementUrl();
		}

		return UriComponentsBuilder.fromUriString(getManagementBaseUrl()).path("/").path(getEndpointsWebPath())
				.toUriString();
	}

	protected String getManagementBaseUrl() {
		String baseUrl = this.instance.getManagementBaseUrl();

		if (StringUtils.hasText(baseUrl)) {
			return baseUrl;
		}

		if (isManagementPortEqual()) {
			return this.getServiceUrl();
		}

		Ssl ssl = (this.management.getSsl() != null) ? this.management.getSsl() : this.server.getSsl();
		return UriComponentsBuilder.newInstance().scheme(getScheme(ssl)).host(getManagementHost())
				.port(getLocalManagementPort()).toUriString();
	}

	protected boolean isManagementPortEqual() {
		return this.localManagementPort == null || this.localManagementPort.equals(this.localServerPort);
	}

	protected String getEndpointsWebPath() {
		return this.webEndpoint.getBasePath();
	}

	protected String getHealthUrl() {
		if (this.instance.getHealthUrl() != null) {
			return this.instance.getHealthUrl();
		}
		return UriComponentsBuilder.fromHttpUrl(getManagementBaseUrl()).path("/").path(getHealthEndpointPath())
				.toUriString();
	}

	protected Map<String, String> getMetadata() {
		Map<String, String> metadata = new LinkedHashMap<>();
		metadata.putAll(this.metadataContributor.getMetadata());
		metadata.putAll(this.instance.getMetadata());
		return metadata;
	}

	protected String getServiceHost() {
		InetAddress address = this.server.getAddress();
		if (address == null) {
			address = getLocalHost();
		}
		return getHost(address);
	}

	protected String getManagementHost() {
		InetAddress address = this.management.getAddress();
		if (address != null) {
			return getHost(address);
		}
		return getServiceHost();
	}

	protected InetAddress getLocalHost() {
		try {
			return InetAddress.getLocalHost();
		}
		catch (UnknownHostException ex) {
			throw new IllegalArgumentException(ex.getMessage(), ex);
		}
	}

	protected Integer getLocalServerPort() {
		if (this.localServerPort == null) {
			throw new IllegalStateException(
					"couldn't determine local port. Please set spring.boot.admin.client.instance.service-base-url.");
		}
		return this.localServerPort;
	}

	protected Integer getLocalManagementPort() {
		if (this.localManagementPort == null) {
			return this.getLocalServerPort();
		}
		return this.localManagementPort;
	}
```