# Creating a Model Listeners

This tutorial will show you how to add a custom model listener by implementing the [ModelListener](https://github.com/liferay/liferay-portal/blob/7.3.0-ga1/portal-kernel/src/com/liferay/portal/kernel/model/ModelListener.java) interface.

Model listeners listen for persistence method calls that signal changes to a specified model (such as `update` or `add` methods). You can define model listeners for out-of-the-box entities (like `JournalArticle` or `AssetEntry`), or for custom entities.

Most of the methods model listeners use are called from DXP's [BasePersistenceImpl](https://github.com/liferay/liferay-portal/blob/7.3.0-ga1/portal-kernel/src/com/liferay/portal/kernel/service/persistence/impl/BasePersistenceImpl.java) class.

## Overview

1. [Deploy an Example](#deploy-an-example)
1. [Walk Through the Example](#walk-through-the-example)
1. [Additional Information](#additional-information)

## Deploy an Example

In this section, we will deploy an example model listener for the `JournalArticle` model on your instance of Liferay DXP. Follow these steps:

1. Start up the DXP image.

```
docker run -it -p 8080:8080 liferay/portal:7.3.0-ga1
```

1. Download and unzip `Acme Model Listener`.

```
curl https://learn.liferay.com/dxp-7.x/liferay-internals/extending-liferay/liferay-n4g6.zip -O
```

```
unzip liferay-n4g6.zip
```

1. Build and deploy the example.

```
./gradlew deploy -Ddeploy.docker.container.id=$(docker ps -lq)
```

```note::
   This command is the same as copying the deployed jars to /opt/liferay/osgi/modules on the Docker container.
```

1. Confirm the deployment in the Docker container console.

```
STARTED com.acme.n4g6.impl_1.0.0
```

1. Verify that the example model listener was added by viewing the added log message. Open your browser to `https://localhost:8080` and navigate to the Site menu → _Content & Data_ → _Web Content_.

	![The web content administration page in the Site menu.](./creating-a-model-listener/images/01.png)

	Click the add ![Add icon](../../images/icon-add.png) button, then click _Basic Web Content_ to add a new article. Fill out a title and some content, then click _Publish_. A warning message appears in the console:

	```
	2020-03-17 23:14:56.301 WARN  [http-nio-8080-exec-5][N4G6ModelListener:23] A new web content article was added.
	```

Congratulations, you've successfully built and deployed a new model listener that implements `ModelListener`.

Next, let's dive deeper to learn more.

## Walk Through the Example

In this section, we will review the example we deployed. First, we will annotate the class for OSGi registration. Second, we will review the `ModelListener` interface. And third, we will complete our implementation of `ModelListener`.

### Annotate the Class for OSGi Registration

```java
@Component(immediate = true, service = ModelListener.class)
public class N4G6ModelListener extends BaseModelListener<JournalArticle> {
```

> Define the class to listen to events for (in this example, `JournalArticle`). When this model listener is registered, it will listen to events for the model defined. The model can be an out-of-the-box entity or a custom entity.
>
> Extending the `BaseModelListener` class gives a default, empty implementation for each of `ModelListener`'s methods. This allows you to only override the methods you need.

### Review the `ModelListener` Interface

All of the `ModelListener` interface's methods are optional to override if you extend the `BaseModelListener` class.

You can provide your own implementation for any of the following methods:

```java
public void onAfterAddAssociation(
		Object classPK, String associationClassName,
		Object associationClassPK)
	throws ModelListenerException;
```

> This method is called after an entity related to the model listener's associated entity is added. These method calls may appear in different places, depending on what models have relationships with your listener's model.

```java
public void onAfterCreate(T model) throws ModelListenerException;
```

> This method is called after a new instance of the model listener's associated entity is added. This happens in `BasePersistenceImpl`'s [update](https://github.com/liferay/liferay-portal/blob/7.3.0-ga1/portal-kernel/src/com/liferay/portal/kernel/service/persistence/impl/BasePersistenceImpl.java#L524) method.

```java
public void onAfterRemove(T model) throws ModelListenerException;
```

> This method is called after an instance of the model listener's associated entity is removed. This happens in `BasePersistenceImpl`'s [remove](https://github.com/liferay/liferay-portal/blob/7.3.0-ga1/portal-kernel/src/com/liferay/portal/kernel/service/persistence/impl/BasePersistenceImpl.java#L460) method.

```java
public void onAfterRemoveAssociation(
		Object classPK, String associationClassName,
		Object associationClassPK)
	throws ModelListenerException;
```

> This method is called after an entity related to the model listener's associated entity is removed. These method calls may appear in different places, depending on what models have relationships with your relationship's model.

```java
public void onAfterUpdate(T model) throws ModelListenerException;
```

> This method is called after an instance of the model listener's associated entity is updated. This happens in `BasePersistenceImpl`'s [update](https://github.com/liferay/liferay-portal/blob/7.3.0-ga1/portal-kernel/src/com/liferay/portal/kernel/service/persistence/impl/BasePersistenceImpl.java#L524) method.

```java
public void onBeforeAddAssociation(
		Object classPK, String associationClassName,
		Object associationClassPK)
	throws ModelListenerException;
```

> This method is called before an entity related to the model listener's associated entity is added. These method calls may appear in different places, depending on what models have relationships with your listener's model.

```java
public void onBeforeCreate(T model) throws ModelListenerException;
```

> This method is called before a new instance of the model listener's associated entity is added. This happens in `BasePersistenceImpl`'s [update](https://github.com/liferay/liferay-portal/blob/7.3.0-ga1/portal-kernel/src/com/liferay/portal/kernel/service/persistence/impl/BasePersistenceImpl.java#L524) method.

```java
public void onBeforeRemove(T model) throws ModelListenerException;
```

> This method is called before an instance of the model listener's associated entity is removed. This happens in `BasePersistenceImpl`'s [remove](https://github.com/liferay/liferay-portal/blob/7.3.0-ga1/portal-kernel/src/com/liferay/portal/kernel/service/persistence/impl/BasePersistenceImpl.java#L460) method.

```java
public void onBeforeRemoveAssociation(
		Object classPK, String associationClassName,
		Object associationClassPK)
	throws ModelListenerException;
```

> This method is called before an entity related to the model listener's associated entity is removed. These method calls may appear in different places, depending on what models have relationships with your relationship's model.

```java
public void onBeforeUpdate(T model) throws ModelListenerException;
```

> This method is called before an instance of the model listener's associated entity is updated. This happens in `BasePersistenceImpl`'s [update](https://github.com/liferay/liferay-portal/blob/7.3.0-ga1/portal-kernel/src/com/liferay/portal/kernel/service/persistence/impl/BasePersistenceImpl.java#L524) method.

### Complete the Model Listener

The model listener only needs custom logic for the events that are important to you.

This example only needs logic after a new instance of the model listener's associated entity is created. This means the logic must be added to the `onAfterCreate` method:

```java
public void onAfterCreate(JournalArticle model)
	throws ModelListenerException {

	if (_log.isWarnEnabled()) {
		_log.warn("A new web content article was added.");
	}
}
```

## Conclusion

Congratulations! You now know the basics for implementing the `ModelListener` interface, and have added a new model listener to Liferay DXP.