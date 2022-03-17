---
date: 2022-3-10
title: SpringBoot核心运行原理
tags: SpringBoot
categories: Java学习
cover: /img/cover/springboot_1.png
---

# SpringBoot核心运行原理

```java
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}


	protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    //检查自动配置功能是否开启，默认开启
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
    //通过spring.factories文件中针对EnableAutoConfiguration的注册配置类
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    //对获得的注册配置类集合进行去重处理，防止多个项目引入同样的配置类
		configurations = removeDuplicates(configurations);
    //获取注解中被exclude或excludeName所排除的类的集合
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    //检查被排除类是否可以实例化，是否被自动注册配置所使用，不符合条件则抛出异常
		checkExcludedClasses(configurations, exclusions);
    //从自动配置类集合中去除被排除的类
		configurations.removeAll(exclusions);
    //检查配置类的注解是否符合spring.factories文件中AutoConfigurationImportFilter指定的注解检查条件
		configurations = getConfigurationClassFilter().filter(configurations);
    //将筛选完成的配置类和排查的配置类构建为事件类，并传入监听器。监听器的配置在spring.factories文件中，通过AutoConfigurationImportListener指定
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}

	protected boolean isEnabled(AnnotationMetadata metadata) {
		if (getClass() == AutoConfigurationImportSelector.class) {
			return getEnvironment().getProperty(EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class, true);
		}
		return true;
	}

```

