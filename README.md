Caldroid
========

Caldroid is a fragment that display calendar with dates in a month. Caldroid can be used as embedded fragment, or as dialog fragment. User can also swipe left/right to navigate to different months.

Caldroid is fully localized. Client can customize start day of the week for different countries. By default calendar start on Sunday.

Caldroid can be used with Android 2.2 and above. Caldroid is extracted from [official Roomorama application](https://play.google.com/store/apps/details?id=com.roomorama)

If you are using Caldroid in your app and keen to list it here, please open a new issue on Github :)


<img src="https://raw.github.com/thomasdao/Caldroid/master/screenshot/1.png" width="270" style="margin-right:10px;">
<img src="https://raw.github.com/thomasdao/Caldroid/master/screenshot/2.png" width="270">

Setup
=====
Just clone the repo and check out the CaldroidSample to see how the library works.

To use in your project, just reference the child Caldroid project as a project library. If you see JAR mismatched error, just replace your android-support-v4.jar to the jar inside Caldroid. Make sure you compile the project against Android 4.2 and above. This is to allow nested fragment. See more at http://developer.android.com/about/versions/android-4.2.html#NestedFragments


Features
========

##Flexible setup: can be embedded or shown as dialog
To embed the caldroid fragment, use below code:

```
CaldroidFragment caldroidFragment = new CaldroidFragment();
Bundle args = new Bundle();
Calendar cal = Calendar.getInstance();
args.putInt("month", cal.get(Calendar.MONTH) + 1);
args.putInt("year", cal.get(Calendar.YEAR));
caldroidFragment.setArguments(args);

FragmentTransaction t = getSupportFragmentManager().beginTransaction();
t.add(R.id.calendar1, caldroidFragment);
t.commit();
```

Caldroid accepts numerous arguments during start up: 

```
int month
int year
String dialogTitle
boolean showNavigationArrows
ArrayList<String> disableDates
ArrayList<String> selectedDates
String minDate
String maxDate with yyyy-MM-dd format
boolean enableSwipe
int startDayOfWeek
```

To customize the startDayOfWeek, just use 

```
Bundle args = new Bundle();
args.putInt("startDayOfWeek", 6); // calendar starts on SATURDAY
caldroidFragment.setArguments(args);
```

Caldroid follows the same convention as the JODA date time constants:

```
MONDAY: 1
TUESDAY: 2
WEDNESDAY: 3
THURSDAY: 4
FRIDAY: 5
SATURDAY: 6
SUNDAY: 7
```


To show the caldroid fragment as a dialog, you might want to set the dialog title. There is a convenient method for that:

```
CaldroidFragment dialogCaldroidFragment = CaldroidFragment.newInstance("Select a date", 3, 2013);
dialogCaldroidFragment.show(getSupportFragmentManager(),"TAG");
```

## Set min / max date

Client can use below methods: 

```
setMinDate(Date minDate)
setMinDateFromString(String minDateString, String dateFormat)

setMaxDate(Date minDate)
setMaxDateFromString(String maxDateString, String dateFormat)
```

To refresh the calendar, just call ```refreshView()```

## Set disabled dates

Client can either provide ArrayList<Date> or ArrayList<String> to Caldroid.
```
setDisableDates(ArrayList<Date> disableDateList)
setDisableDatesFromString(ArrayList<String> disableDateStrings)
setDisableDatesFromString(ArrayList<String> disableDateStrings, String dateFormat)
```

To clear the disabled dates:
```
clearDisableDates()
```

##Select dates within a range
To select dates within a range:

```
setSelectedDates(Date fromDate, Date toDate)
setSelectedDateStrings(String fromDateString, String toDateString, String dateFormat)
```

To clear the selected dates:
```
clearSelectedDates()
```


##Show / Hide the navigation arrows to move to previous or next month
To show/hide the navigation arrows:

```
setShowNavigationArrows(boolean showNavigationArrows)
```

To enable / disable swipe:

```
setEnableSwipe(boolean enableSwipe)
```

Client can programmatically move the calendar (with animation) to a specified date:

```
public void moveToDate(Date date);
public void moveToDateTime(DateTime dateTime);
```

##Allow user to select a date and inform listener

Caldroid inform clients via CaldroidListener. 

```
CaldroidListener listener = new CaldroidListener() {

	@Override
	public void onSelectDate(Date date, View view) {
		Toast.makeText(getApplicationContext(), formatter.format(date),
				Toast.LENGTH_LONG).show();
	}

	@Override
	public void onChangeMonth(int month, int year) {
		String text = "month: " + month + " year: " + year;
		Toast.makeText(getApplicationContext(), text, Toast.LENGTH_LONG)
				.show();
	}

};
    
caldroidFragment.setCaldroidListener(listener);

```

##Handle screen rotation

To handle rotation properly, Caldroid provides method to get current states of the fragment:

```
public Bundle getSavedStates();
public void saveStatesToKey(Bundle outState, String key);
public void restoreStatesFromKey(Bundle savedInstanceState, String key);
public void restoreDialogStatesFromKey(FragmentManager manager, Bundle savedInstanceState, String key, String dialogTag)
```

Using above method, you can save current state of Caldroid on ```onSaveInstanceState(Bundle outState)``` method.

On your activity code:

```
@Override
protected void onSaveInstanceState(Bundle outState) {
	// TODO Auto-generated method stub
	super.onSaveInstanceState(outState);

	if (caldroidFragment != null) {
		caldroidFragment.saveStatesToKey(outState, "CALDROID_SAVED_STATE");
	}

	if (dialogCaldroidFragment != null) {
		dialogCaldroidFragment.saveStatesToKey(outState,
				"DIALOG_CALDROID_SAVED_STATE");
	}
}
```

Then you can restore the state in ```onCreate(Bundle savedInstanceState)``` of your activity. The algorithm is like below:

```
// If Activity is created after rotation
		if (savedInstanceState != null) {
			caldroidFragment.restoreStatesFromKey(savedInstanceState,
					"CALDROID_SAVED_STATE");
		}
		// If activity is created from fresh
		else {
			Bundle args = new Bundle();
			Calendar cal = Calendar.getInstance();
			args.putInt("month", cal.get(Calendar.MONTH) + 1);
			args.putInt("year", cal.get(Calendar.YEAR));
			args.putBoolean("enableSwipe", true);
			args.putBoolean("fitAllMonths", false);
			caldroidFragment.setArguments(args);
		}

```

If you use Caldroid as dialog, you can use ```restoreDialogStatesFromKey```

```
    final String dialogTag = "CALDROID_DIALOG_FRAGMENT";
    if (savedInstanceState != null) {
      dialogCaldroidFragment.restoreDialogStatesFromKey(getSupportFragmentManager(),
      					savedInstanceState, "DIALOG_CALDROID_SAVED_STATE",
      					dialogTag);
			Bundle args = dialogCaldroidFragment.getArguments();
			args.putString("dialogTitle", "Select a date");
		} else {
			// Setup arguments
			Bundle bundle = new Bundle();
			// Setup dialogTitle
			bundle.putString("dialogTitle", "Select a date");
			dialogCaldroidFragment.setArguments(bundle);
		}
```

Refer to the CaldroidSampleActivity for more detail.

##Allow customized cell for the dates gridView

Caldroid provides flexible API to supply your own cell view. What you have to do is:

1) Create your own cell view layout in your project

2) Subclass CaldroidGridAdapter and override ```getView(int position, View convertView, ViewGroup parent)```. See CaldroidSampleCustomAdapter.java for more detail. Here you can customize everything: layout, text color, background for different states (normal, disable, selected)

3) Subclass CaldroidFragment to use your custom adapter instead of the default CaldroidGridAdapter. This is simplest step:

```
public class CaldroidSampleCustomFragment extends CaldroidFragment {

	@Override
	public CaldroidGridAdapter getNewDatesGridAdapter(int month, int year) {
		// TODO Auto-generated method stub
		return new CaldroidSampleCustomAdapter(getActivity(), month, year, getCaldroidData(), extraData);
	}

}
```

4) Use your new customized fragment in your project instead of the default CaldroidFragment.

To see how it works, you can uncomment this line in CaldroidSampleActivity

```
// final CaldroidSampleCustomFragment caldroidFragment = new CaldroidSampleCustomFragment();
```

The ```extraData``` is a ```HashMap<String, Object>```, is designed to let client injects custom data to CaldroidGridAdapter, so that data can be used to customize the date grid view. Usage is simple:

In your client code:

```
// To set the extraData:
HashMap<String, Object> extraData = caldroidFragment.getExtraData();
extraData.put("YOUR_CUSTOM_DATA_KEY1", yourCustomData1);
extraData.put("YOUR_CUSTOM_DATA_KEY2", yourCustomData2);

// Refresh view
caldroidFragment.refreshView();

```

In the CaldroidSampleCustomAdapter:

```
@Override
	public View getView(int position, View convertView, ViewGroup parent) {
    // Get your data here
    ArrayList yourCustomData1 = (ArrayList) extraData.get("YOUR_CUSTOM_DATA_KEY1");
    String yourCustomData2 = (String) extraData.get("YOUR_CUSTOM_DATA_KEY2");
    
    // Continue to build your customized view
  }
```

## Client can customize look and feel of almost all views

Client can set color of the weekday symbols (SUN, MON, ...) by:

```
WeekdayArrayAdapter.textColor = Color.BLUE;
```

For more customization, client can supply adapter to the weekdayGridView

```
caldroidFragment.getWeekdayGridView().setAdapter(YOUR_ADAPTER);
```

User can also customize the navigation arrows and month title textView

```
public Button getLeftArrowButton();
public Button getRightArrowButton();
public TextView getMonthTitleTextView();
```

Basic Structure
===============

Caldroid fragment includes 4 main parts:
  
1) Month title view: show the month and year (e.g MARCH, 2013)
  
2) Navigation arrows: to navigate to next month or previous month
  
3) Weekday gridview: contains only 1 row and 7 columns. To display
  "SUN, MON, TUE, WED, THU, FRI, SAT"
  
4) An infinite view pager that allow user to swipe left/right to change month. This library is taken from https://github.com/antonyt/InfiniteViewPager

This infinite view pager recycles 4 fragment, each fragment contains a gridview with 7 columns to display the dates in month. Whenever user swipes different screen, the date grid views are updated.
  

Others
======
  
Caldroid code is simple and clean partly because of powerful JODA DateTime library!


License
=======
See LICENSE.md

