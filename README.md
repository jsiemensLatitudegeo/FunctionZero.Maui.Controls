# ~~Breaking news (literally)~~
~~It seems the indent of tree nodes is broken after updating to VS 17.4.0 Preview 1.0.  
I will update this text when things change.~~  
Everything is [fixed](https://www.nuget.org/packages/FunctionZero.Maui.Controls) :smile:  
# Controls
[NuGet package](https://www.nuget.org/packages/FunctionZero.Maui.Controls)

## TreeViewZero
![Sample image](https://github.com/Keflon/FunctionZero.Maui.Controls/blob/master/AndroidTree.png?raw=true)

This control allows you to visualise a tree of any data. Each *trunk* node must provide its children using a public property that supports the IEnumerable interface.  
If the children are in a collection that supports `INotifyCollectionChanged` the control will track changes to the underlying tree data.  
Children are lazy-loaded and the UI is virtualised.  

### TreeViewZero exposes the following properties
Property | Type | Bindable | Purpose
:----- | :---- | :----: | :-----
IndentMultiplier        | double           | Yes (OneTime) | How far the TreeNode should be indented for each nest level. Default is 15.0
IsRootVisible           | bool             | Yes  | Specifies whether the root node should be shown or omitted.
ItemContainerStyle      | Style            | Yes  | An optional `Style` that can be applied to the `TreeNodeZero` objects that represent each node.
ItemHeight              | float            | Yes  | The height of each row in the TreeView
ItemsSource             | object           | Yes  | Set this to your root node  
ScrollOffset            | float            | YES! | This is the absolute offset of the scroller
SelectedItem            | object           | Yes  | Set to the currently selected item, i.e. an instance of your *ViewModel* data, or null
SelectedItems           | IList            | Yes  | All currently selected items. Default is an `ObservableCollection<object>`. You can bind to it or set your own collection, and if it supports `INotifyCollectionChanged` the `TreeViewZero` will track it.
SelectionMode           | SelectionMode    | Yes  | Alloows a `SelectionMode` of None, Single or Multiple.
TreeChildren            | IEnumerable      | No   | This is exposed for future capabilities and exposes all items *potentially* visible in the viewport.
TreeItemControlTemplate | ControlTemplate  | Yes  | Alows you to replace the default `ControlTemplate` used to render each node
TreeItemTemplate        | TemplateProvider | Yes  | Used to draw the data for each node. Set this to a `TreeItemDataTemplate` or a `TreeItemDataTemplateSelector`. See below.

### TreeItemDataTemplate
`TreeItemDataTemplate` tells a tree-node how to draw its content, how to get its children and whether it should bind `IsExpanded` to the underlying data.  
It declares the following properties:

Property | Type | Purpose
:----- | :---- | :-----
ChildrenPropertyName    | string       | The name of the property used to find the node children
IsExpandedPropertyName  | string       | The name of the property used to store whether the node is expanded
ItemTemplate            | DataTemplate | The DataTemplate used to draw this node
TargetType              | Type         | When used in a `TreeItemDataTemplateSelector`, identifies the least-derived nodes the ItemTemplate can be applied to.

### Create a TreeViewZero

Given a hierarchy of `MyNode`
```csharp
public class MyNode
{
   public string Name { get; set;}
   public IEnumerable<MyNode> MyNodeChildren { get; set; }
}
```

Add the namespace:
```xml
xmlns:cz="clr-namespace:FunctionZero.Maui.Controls;assembly=FunctionZero.Maui.Controls"
```

Then declare a `TreeViewZero` like this:
```xml
<!--Tip: A generous ItemHeight ensures the chevrons aren't too small to tap with your finger-->
<cz:TreeViewZero ItemsSource="{Binding RootNode}" ItemHeight="100">
    <cz:TreeViewZero.TreeItemTemplate>
        <cz:TreeItemDataTemplate ChildrenPropertyName="MyNodeChildren">
            <DataTemplate>
                <Label Text="{Binding Name}" />
            </DataTemplate>
        </cz:TreeItemDataTemplate>
    </cz:TreeViewZero.TreeItemTemplate>
</cz:TreeViewZero>
```

## Tracking changes in the data
If the children of a node support `INotifyCollectionChanged`, the TreeView will track all changes automatically.  
If the properties on your node support `INotifyPropertyChanged` then they too will be tracked.  

For example, TreeViewZero will track changes to `Name`, `IsExpanded` and any 
modifications to the `Children` collection on the following node:
```csharp
public class MyObservableNode : BaseClassWithInpc
{
   private string _name;
   public string Name
   {
      get => _name;
      set => SetProperty(ref _name, value);
   }

   private bool _isMyNodeExpanded;
   public bool IsMyNodeExpanded
   {
      get => _isMyNodeExpanded;
      set => SetProperty(ref _isMyNodeExpanded, value);
   }

   public ObservableCollection<MyObservableNode> Children {get; set;}
}
```
This is how to bind the `IsMyNodeExpanded` from our data, to `IsExpanded` on the TreeNode ...

```xml
<cz:TreeViewZero.TreeItemTemplate>
    <cz:TreeItemDataTemplate ChildrenPropertyName="Children" IsExpandedPropertyName="IsMyNodeExpanded">
        <DataTemplate>
            ...
        </DataTemplate>
    </cz:TreeItemDataTemplate>
</cz:TreeViewZero.TreeItemTemplate>
```
### TreeItemDataTemplateSelector
If your tree of data consists of disparate nodes with different properties for their `Children`, 
use a `TreeItemDataTemplateSelector` and set `TargetType` for each `TreeItemDataTemplate`.  

Note: In this example, the tree data can contain nodes of type `LevelZero`, `LevelOne` and `LevelTwo` where each type has a different property to provide its children.  

The first `TargetType` your data-node can be assigned to is used. Put another way, the first `TargetType` the data-node can be cast to, wins.
```xml
<cz:TreeViewZero ItemsSource="{Binding SampleTemplateTestData}" ItemHeight="20" >
    <cz:TreeViewZero.TreeItemTemplate>
        <cz:TreeItemDataTemplateSelector>
            <cz:TreeItemDataTemplate ChildrenPropertyName="LevelZeroChildren" TargetType="{x:Type test:LevelZero}" IsExpandedPropertyName="IsLevelZeroExpanded">
                <DataTemplate>
                    <Label Text="{Binding Name}" BackgroundColor="Yellow" />
                </DataTemplate>
            </cz:TreeItemDataTemplate>

            <cz:TreeItemDataTemplate ChildrenPropertyName="LevelOneChildren" TargetType="{x:Type test:LevelOne}" IsExpandedPropertyName="IsLevelOneExpanded">
                <DataTemplate>
                    <Label Text="{Binding Name}" BackgroundColor="Cyan" />
                </DataTemplate>
            </cz:TreeItemDataTemplate>

            <cz:TreeItemDataTemplate ChildrenPropertyName="LevelTwoChildren" TargetType="{x:Type test:LevelTwo}" IsExpandedPropertyName="IsLevelTwoExpanded">
                <DataTemplate>
                    <Label Text="{Binding Name}" BackgroundColor="Pink" />
                </DataTemplate>
            </cz:TreeItemDataTemplate>

            <cz:TreeItemDataTemplate ChildrenPropertyName="LevelThreeChildren" TargetType="{x:Type test:LevelThree}" IsExpandedPropertyName="IsLevelThreeExpanded">
                <DataTemplate>
                    <Label Text="{Binding Name}" BackgroundColor="Crimson" />
                </DataTemplate>
            </cz:TreeItemDataTemplate>
        </cz:TreeItemDataTemplateSelector>
    </cz:TreeViewZero.TreeItemTemplate>
</cz:TreeViewZero>
```
### Customising TreeItemDataTemplateSelector
If you want **full-control** over the `TreeItemTemplate` per node, you can easily implement your own 
`TreeItemDataTemplateSelector` and override `OnSelectTemplate`. Here's an example that chooses a template 
based on whether the node has children or not:

```csharp
public class MyTreeItemDataTemplateSelector : TemplateProvider
{
    /// These should be set in the xaml markup. (or code-behind, if that's how you roll)
    public TreeItemDataTemplate TrunkTemplate{ get; set; }
    public TreeItemDataTemplate LeafTemplate{ get; set; }

    public override TreeItemDataTemplate OnSelectTemplateProvider(object item)
    {
        if(item is MyTreeNode mtn)
            if((mtn.Children != null) && (mtn.Children.Count != 0))
                return TrunkTemplate;
        
        return LeafTemplate;
    }
}
```
Take a look at [TreeItemDataTemplateSelector.cs](https://github.com/Keflon/FunctionZero.Maui.Controls/blob/master/FunctionZero.Maui.Controls/TreeItemDataTemplateSelector.cs) 
for an example of how to provide a *collection* of `TreeItemDataTemplate` instances to your TemplateProvider.

## Drawing your own tree-nodes
Do this if you want to change the way the whole tree-node is drawn, e.g. to replace the *chevron*. 
It is a two-step process.
1. Create a `ControlTemplate` for the node
1. Apply it to the `TreeViewZero`

The *templated parent* for the `ControlTemplate` is a `ListItemZero`. It exposes these properties:  


Property    | Type   | Purpose
:----- | :----: | :-----
IsPrimary    | bool         | If selection is allowed, this tracks the current `SelectedItem`
IsSelected   | bool         | If the current item is selected, this is true. Note we can have multiple items *selected*, but only one *SelectedItem*
ItemIndex    | int          | For internal use when managing the cache
ItemTemplate | DataTemplate | The `DataTemplate` used to generate the `ListViewItem` Content

The `BindingContext` of the *templated parent* is a [TreeNodeContainer](https://github.com/Keflon/FunctionZero.TreeListItemsSourceZero) and includes the following properties: 

Property    | Type   | Purpose
:----- | :----: | :-----
Indent      | int    | How deep the node should be indented. It is equal to `NestLevel`, or `NestLevel-1` if the Tree Root is not shown.
NestLevel   | int    | The depth of the node in the data.
IsExpanded  | bool   | This property reflects whether the TreeNode is expanded.
ShowChevron | bool   | Whether the chevron is drawn. True if the node has children.
Data        | object | This is the tree-node data for this TreeNodeZero instance, i.e. your data!



### Step 1 - Create a `ControlTemplate` ...

You can base the `ControlTemplate` on the default, show here, or bake your own entirely.  
```xml
<ControlTemplate x:Key="defaultControlTemplate">
    <HorizontalStackLayout HeightRequest="{Binding Height, Mode=OneWay, Source={x:Reference tcp}}" >
        <!--
        The ControlTemplate sets the TreeNodeSpacer BindingContext to "{Binding Source={RelativeSource TemplatedParent}}" for us
        i.e. a ListItemZero.
        The TreeNodeSpacer walks up the visual-tree to find the containing TreeViewZero, to get the IndentMultiplier.
        It then sets its WidthRequest to the IndentMultiplier * (ListItemZero.BindingContext.Indent - 1)
        -->
        <cz:TreeNodeSpacer />

        <cz:Chevron
            IsExpanded="{TemplateBinding BindingContext.IsExpanded, Mode=TwoWay}" 
            ShowChevron="{TemplateBinding BindingContext.ShowChevron, Mode=TwoWay}" 
            />
        <!--This is simply a ContentPresenter that allows us to specify a BindingContext for the Content-->
        <cz:TreeContentPresenter 
            VerticalOptions="Fill" 
            x:Name="tcp" 
            HorizontalOptions="Fill"   
            BindingContext="{TemplateBinding BindingContext.Data, Mode=OneWay}"
            />
    </HorizontalStackLayout>
</ControlTemplate>
```
 

### Step 2 - give it to the TreeView ...
```xml
<cz:TreeViewZero
    ItemsSource="{Binding SampleData}" 
    IndentMultiplier="20" 
    TreeItemControlTemplate="{StaticResource MyTreeItemControlTemplate}"
    ItemHeight="60"
    >
    <cz:TreeViewZero.TreeItemTemplate>
        <cz:TreeItemDataTemplate ChildrenPropertyName="Children" IsExpandedPropertyName="IsDataExpanded">
            <DataTemplate>
                <Label Text="{Binding Name}" BackgroundColor="Pink" />
            </DataTemplate>
        </cz:TreeItemDataTemplate>
    </cz:TreeViewZero.TreeItemTemplate>
</cz:TreeViewZero>
```

## Known issues:
~~There are two known issues, both in the **WinUI** platform, both relating to the underlying `CollectionView` used by `TreeViewZero`.~~ 
~~1. The `CollectionView` has a minimum item-spacing bug, reported [here](https://github.com/dotnet/maui/issues/4520)~~
~~2. The `CollectionView` is not recycling containers, reported [here](https://github.com/dotnet/maui/issues/8151)~~  

~~I'll update the source and [NuGet package](https://www.nuget.org/packages/FunctionZero.Maui.Controls) once these bugs are fixed, if necessary.~~

To work around current bugs, this no longer uses the MAUI CollectionView or ListView.

  
