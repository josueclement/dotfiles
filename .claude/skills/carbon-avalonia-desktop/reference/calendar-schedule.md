# CalendarSchedule

Week/month appointment view in `Carbon.Avalonia.Desktop.Controls.CalendarSchedule`
(`xmlns:controls="using:Carbon.Avalonia.Desktop.Controls.CalendarSchedule"`). Bind an
`IEnumerable<CalendarScheduleItem>`; week view supports drag-move and drag-resize (15-min snap).

## Public API

**`CalendarSchedule`** (`TemplatedControl`):
- `Items` : `IEnumerable<CalendarScheduleItem>?`
- `DisplayDate` : `DateTimeOffset` (TwoWay, default now)
- `ViewMode` : `CalendarViewMode` (`Week` | `Month`, TwoWay, default `Month`)
- `SelectedDate` : `DateTimeOffset?` (TwoWay), `SelectedItem` : `CalendarScheduleItem?` (TwoWay)
- `FirstDayOfWeek` : `DayOfWeek` (default `Monday`)
- events `ItemMoved` and `ItemResized` : `EventHandler<CalendarScheduleItemChangedEventArgs>`

**`CalendarScheduleItem`** (`ObservableObject` — mutable, observable): `Title` : `string?`,
`Start` / `End` : `DateTimeOffset`, `Color` : `IBrush?`, `Description` : `string?`.

**`CalendarScheduleItemChangedEventArgs`** : `Item`, `OriginalStart`/`OriginalEnd`,
`NewStart`/`NewEnd`.

## Example

```xml
<controls:CalendarSchedule Items="{Binding Items}" ViewMode="Week" Margin="16" />
```

```csharp
using Carbon.Avalonia.Desktop.Controls.CalendarSchedule;

public class SchedulePageViewModel : ObservableObject
{
    public SchedulePageViewModel()
    {
        var monday = DateTimeOffset.Now.Date;   // snap to your week start as needed
        Items = new List<CalendarScheduleItem>
        {
            new()
            {
                Title = "Team Standup",
                Start = monday.AddHours(9),
                End   = monday.AddHours(9.5),
                Color = new SolidColorBrush(Color.Parse("#3574F0")),
                Description = "Daily standup meeting"
            },
            new()
            {
                Title = "Sprint Planning",
                Start = monday.AddDays(1).AddHours(10),
                End   = monday.AddDays(1).AddHours(12),
                Color = new SolidColorBrush(Color.Parse("#E8A33D"))
            },
        };
    }

    public IReadOnlyList<CalendarScheduleItem> Items { get; }
}
```

To persist drag edits, handle `ItemMoved` / `ItemResized` (the item is already mutated; use the args to
log or validate, and revert by resetting `Start`/`End` if you reject the change).

## Common mistakes

- Using `DateTime` instead of `DateTimeOffset` for `Start`/`End`.
- Expecting drag editing in **month** view — move/resize is a **week**-view feature; set
  `ViewMode="Week"`.
