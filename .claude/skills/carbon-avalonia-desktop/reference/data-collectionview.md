# CollectionView (Data)

A WPF-style collection view layer in `Carbon.Avalonia.Desktop.Data`
(`xmlns:carbon="using:Carbon.Avalonia.Desktop.Data"`) for sorting, filtering, and grouping over an
`IEnumerable` that raises `INotifyCollectionChanged` (e.g. `ObservableCollection<T>`), without mutating
the source.

## Key types

- **`CollectionViewSource`** : `Source` : `object?`, `SortDescriptions` (of `SortDescription`),
  `GroupDescriptions` (of `PropertyGroupDescription`), and a `View` you bind list controls to.
- **`SortDescription`** : `PropertyName` : `string`, `Direction` : `SortDirection`
  (`Ascending` | `Descending`).
- **`PropertyGroupDescription`** : `PropertyName` : `string`.
- **`CollectionViewGroup`** : `Key`, `ItemCount`, `Items` — bind to `View.Groups` for grouped display.

## Example — sorted + grouped

Declare the source(s) as resources and bind `ItemsSource` to `View` (flat) or `View.Groups` (grouped):

```xml
<UserControl xmlns:carbon="using:Carbon.Avalonia.Desktop.Data"
             x:DataType="vm:PeopleViewModel" ...>
  <UserControl.Resources>
    <carbon:CollectionViewSource x:Key="SortedPeople" Source="{Binding People}">
      <carbon:CollectionViewSource.SortDescriptions>
        <carbon:SortDescription PropertyName="LastName" Direction="Ascending" />
      </carbon:CollectionViewSource.SortDescriptions>
    </carbon:CollectionViewSource>

    <carbon:CollectionViewSource x:Key="GroupedPeople" Source="{Binding People}">
      <carbon:CollectionViewSource.SortDescriptions>
        <carbon:SortDescription PropertyName="LastName" Direction="Ascending" />
      </carbon:CollectionViewSource.SortDescriptions>
      <carbon:CollectionViewSource.GroupDescriptions>
        <carbon:PropertyGroupDescription PropertyName="Department" />
      </carbon:CollectionViewSource.GroupDescriptions>
    </carbon:CollectionViewSource>
  </UserControl.Resources>

  <StackPanel>
    <!-- Flat, sorted -->
    <ListBox ItemsSource="{Binding View, Source={StaticResource SortedPeople}}" />

    <!-- Grouped -->
    <ItemsControl ItemsSource="{Binding View.Groups, Source={StaticResource GroupedPeople}}">
      <ItemsControl.ItemTemplate>
        <DataTemplate x:DataType="carbon:CollectionViewGroup">
          <StackPanel>
            <TextBlock FontWeight="SemiBold">
              <Run Text="{Binding Key}" /><Run Text=" (" /><Run Text="{Binding ItemCount}" /><Run Text=")" />
            </TextBlock>
            <ItemsControl ItemsSource="{Binding Items}" />
          </StackPanel>
        </DataTemplate>
      </ItemsControl.ItemTemplate>
    </ItemsControl>
  </StackPanel>
</UserControl>
```

The ViewModel just exposes the source collection (add/remove flows through the view live):

```csharp
public ObservableCollection<PersonItem> People { get; } = [ /* … */ ];
public record PersonItem(string FirstName, string LastName, string Department);
```

## Common mistakes

- Source not observable — plain `List<T>` won't update the view on add/remove; use
  `ObservableCollection<T>`.
- Binding a list to the `CollectionViewSource` itself — bind to its **`View`** (or `View.Groups`).
- A `PropertyName` that doesn't match a real property yields empty/incorrect groups or ordering.
