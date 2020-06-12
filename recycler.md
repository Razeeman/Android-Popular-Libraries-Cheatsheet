# RECYCLER

- [Epoxy](#epoxy---)
- [Sources](#sources)



[Content](#recycler)
# Epoxy [![Maven][epoxy-mavenbadge]][epoxy-maven] [![Source][epoxy-sourcebadge]][epoxy-source] ![epoxy-starsbadge]

Library for building complex screens in a RecyclerView created by Airbnb.

```gradle
dependencies {
    implementation 'com.airbnb.android:epoxy:3.11.0'
}
```

Models are automatically generated from custom views or databinding layouts via annotation processing. These models are then used in an EpoxyController to declare what items to show in the RecyclerView.

This abstracts the boilerplate of view holders, diffing items and binding payload changes, item types, item ids, span counts, and more, in order to simplify building screens with multiple view types. Additionally, Epoxy adds support for saving view state and automatic diffing of item changes.

### Epoxy. Basic usage

There are two main components of Epoxy:

- The **EpoxyModels** that describe how your views should be displayed in the RecyclerView.
- The **EpoxyController** where the models are used to describe what items to show and with what data.

Epoxy generates models for you based on your view or layout. Generated model classes are suffixed with an underscore are used directly in your EpoxyController classes.

### Epoxy. Models

Epoxy uses EpoxyModel objects to decide which views to display and how to bind data to them. This is similar to the popular ViewModel pattern. Models also allow you to control other aspects of the view, such as the grid span size, id, and saved state. The RecyclerView concept of stable ideas is built into Epoxy, meaning each EpoxyModel should have a unique id to identify it. This allows diffing and saved state. A model's state is determined by its equals and hashCode implementations, which is based on the value of all of the model's properties. This state is used in diffing to determine when a model has changed so Epoxy can update the view. Each model is typed with the View that it supports. The model implements buildView to create a new instance of that view type. Additionally, getViewType returns an int representing that view type for use in view recycling. When a model's state changes, and the model is bound to a view on screen, Epoxy rebinds the view with only the properties that changed. This partial rebind is more efficient than rebinding the whole view.

**From custom views:**

Add the **@ModelView** annotation on a view class. Then, add a "prop" annotation on each setter method to mark it as a property for the model.

```java
@ModelView(autoLayout = Size.MATCH_WIDTH_WRAP_HEIGHT)
public class HeaderView extends LinearLayout {

  @TextProp
  public void setTitle(CharSequence text) {
    titleView.setText(text);
  }
}
```

A **HeaderViewModel_** is then generated in the same package.

**From databinding:**

Set up your xml layouts like normal:

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="title" type="String" />
    </data>

    <TextView
        android:layout_width="120dp"
        android:layout_height="40dp"
        android:text="@{title}" />
</layout>
```

Then, create a **package-info.java** file in any package and add an EpoxyDataBindingLayouts annotation to declare your databinding layouts.

```java
@EpoxyDataBindingLayouts({R.layout.header_view, ... // other layouts })
package com.airbnb.epoxy.sample;

import com.airbnb.epoxy.EpoxyDataBindingLayouts;
import com.airbnb.epoxy.R;
```

From this layout name Epoxy generates a **HeaderViewBindingModel_**.

**From ViewHolders:**

If you use xml layouts without databinding you can create a model class to do the binding.

```java
@EpoxyModelClass(layout = R.layout.header_view)
public abstract class HeaderModel extends EpoxyModelWithHolder<Holder> {
  @EpoxyAttribute String title;

  @Override
  public void bind(Holder holder) {
    holder.header.setText(title);
  }

  static class Holder extends BaseEpoxyHolder {
    @BindView(R.id.text) TextView header;
  }
}
```

A **HeaderModel_** class is generated that subclasses HeaderModel and implements the model details.

### Epoxy. Controllers

A controller defines what items should be shown in the RecyclerView, by adding the corresponding models in the desired order. The EpoxyController encourages usage similar to the popular Model-View-ViewModel and Model-View-Intent patterns. Data flows in one direction: from your state, to the EpoxyModels, to the views on the RecyclerView. User input to the views triggers callbacks that may update your state and restart the cycle. EpoxyController has a second constructor that takes two Handler instances, one for running model building and one for handling diffing. By default these use the main thread, but they can be changed to allow async work for performance improvement.

The controller's **buildModels** method declares which items to show. You are responsible for calling **requestModelBuild** whenever your data changes, which triggers **buildModels** to run again. Epoxy tracks changes in the models and automatically binds and updates views.

As an example, our PhotoController shows a header, a list of photos, and a loader (if more photos are being loaded). The controller's setData(photos, loadingMore) method is called whenever photos are loaded, which triggers a call to buildModels so models representing the state of the new data can be built.

```java
public class PhotoController extends Typed2EpoxyController<List<Photo>, Boolean> {
    @AutoModel HeaderModel_ headerModel;
    @AutoModel LoaderModel_ loaderModel;

    @Override
    protected void buildModels(List<Photo> photos, Boolean loadingMore) {
      headerModel
          .title("My Photos")
          .description("My album description!")
          .addTo(this);

      for (Photo photo : photos) {
        new PhotoModel()
           .id(photo.id())
           .url(photo.url())
           .addTo(this);
      }

      loaderModel
          .addIf(loadingMore, this);
    }
  }
```

**Integrating with RecyclerView**

Get the backing adapter off the EpoxyController to set up RecyclerView:

```java
MyController controller = new MyController();
recyclerView.setAdapter(controller.getAdapter());

// Request a model build whenever your data changes
controller.requestModelBuild();

// Or if you are using a TypedEpoxyController
controller.setData(myData);
```

If you are using the EpoxyRecyclerView integration is easier.

```java
epoxyRecyclerView.setControllerAndBuildModels(new MyController());

// Request a model build on the recyclerview when data changes
epoxyRecyclerView.requestModelBuild();
```


[Content](#recycler)
# Sources

**Epoxy**

https://github.com/airbnb/epoxy
<br>
https://github.com/airbnb/epoxy/wiki/Configuration



[epoxy-maven]: https://search.maven.org/artifact/com.airbnb.android/epoxy
[epoxy-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.airbnb.android/epoxy/badge.svg
[epoxy-source]: https://github.com/airbnb/epoxy
[epoxy-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[epoxy-starsbadge]: https://img.shields.io/github/stars/airbnb/epoxy
