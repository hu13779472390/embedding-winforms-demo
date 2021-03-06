# embedding-of-demo
This is a demo Windows Forms application that embeds an HTML 5 application and shares data over the OpenFin Runtime Inter Application Bus. The demo leverages the OpenFin Runtime, the Openfin.WinForm EmbeddedView control


![embed](embed.png)


#Getting started with Openfin.WinForm
* Follow the [NuGet](https://www.nuget.org/packages/Openfin.WinForm) install instructions.

### EmbeddedView

Once installed add a reference to Openfin.WinForm:
```js
using System;
using System.Windows.Forms;
using Openfin.WinForm;
namespace WinForms.Test
{
    public partial class Form1 : Form
    {
    ...
```
Now place an EmbeddedView control in the screen:
```js
var OpenFinEmbeddedView = new EmbeddedView();
//Assuming we have a Panel control named panel1
panel1.Controls.Add(OpenFinEmbeddedView);
```

### Runtime Options
The Runtime Options object specifies the OpenFin Runtime being used, options include: target runtime (alpha, beta, 32/64 bits...etc), the ability to use remote debugging or specifiying the RVM location, you can read more about options in our [Docs](https://openfin.co/developers/application-config/):
```js
var runtimeOptions = new Openfin.Desktop.RuntimeOptions
{
    Version = "alpha",
    EnableRemoteDevTools = true,
    RemoteDevToolsPort = 9090
};
```

###Application Options
The Application Options object allows you to configure the OpenFin Application being embedded, options include: name, URL, icon and window options, you can read more about options in our [Docs](https://openfin.co/developers/application-config/):
```js
var appOptions = new Openfin.Desktop.ApplicationOptions("of-chart", 
    "of-chart-uuid", "http://cdn.openfin.co/embed-web/chart.html");
```

###Initialize
The EmbeddedView will need to be initialized with both the RuntimeOptions object and the ApplicationOptions object:
```js
    OpenFinEmbeddedView.Initialize(runtimeOptions, appOptions);
```

###OnReady
To programmatically react to when the EmbeddedView has loaded its content, initialized and is ready to be displayed you can subscribe to the OnReady event:
```js
OpenFinEmbeddedView.OnReady += (sender, e) =>
{
    //Any Interactions with the UI must be done in the right thread.
    Utils.InvokeOnUiThreadIfRequired(this, () => 
        textBlockReady.Text = "OpenFinEmbeddedView is ready");
}
```

###Embedding Child Windows
The OpenFinEmbeddedView allows you to embed web applicatons, these will have their own render process and sandbox, but it also allows you to embed child windows that can share the same render process and sandbox, adding the risk of one window crashing the other but using less resources.
```js
OpenFinEmbeddedView.OnReady += (sender, e) =>
{
    //We need to create our WindowOptions Object
    var windowOptions = new WindowOptions("jsdocs", "http://cdn.openfin.co/jsdocs/3.0.1.5/");

    //Assuming we have added a second EmbeddedView called OpenFinEmbeddedViewChild we initialize it.
    OpenFinEmbeddedViewChild.Initialize(runtimeOptions, OpenFinEmbeddedView.OpenfinApplication, windowOptions)
}
```

###Runtime Object
Every EmbeddedView control that shares a RuntimeOptions object will share a connection to the OpenFin Runtime. You can obtain this singleton object via the Runtime.GetRuntimeInstance function. It allows you to publish and subscribe to Inter Application Bus messages, react to disconnect events, and initiate connect calls (this is optional and unnecessary in the case where one or more EmbeddedView control has been initialized).
```js
var openFinRuntime = Runtime.GetRuntimeInstance(runtimeOptions);
openFinRuntime.Connect(() => 
{
    //Any Interactions with the UI must be done in the right thread.
    Utils.InvokeOnUiThreadIfRequired(this, () => 
        textBlockConnected.Text = "OpenFin Runtime is connected");

    //subscribe to hello-from-bus messages from any application
    openFinRuntime.InterApplicationBus.subscribe("*", "hello-from-bus", (senderUuid, topic, data) =>
    {
        var dataAsJObject = JObject.FromObject(data);
        Utils.InvokeOnUiThreadIfRequired(this, () =>
            textBlockMessage.Text = dataAsJObject.GetValue("message"));
    });
});
```
