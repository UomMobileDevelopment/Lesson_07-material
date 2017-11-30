# Lesson 07

## Οδηγίες ασφαλούς χρήσης του Open Maps Weather API key.

Όπως είπαμε και σήμερα στο μάθημα, ΔΕΝ πρέπει να δημοσιεύσετε το κλειδί του OpenWeatherMaps API στον κώδικα της εργασίας σας που θα ανεβεί 
δημόσια στο GitHub.

Θα πρέπει να βάλετε το API ΚΕΥ σε ένα αρχείο στον φάκελο /user/.gradle του υπολογιστή σας και να το διαβάζετε απο εκεί.

```
1. Προσθέστε μια γραμμή στο αρχείο ~/.gradle/gradle.properties  (αν δεν υπάρχει ήδη, δημιουργήστε το), 
η γραμμή θα πρέπει να έχει τη μορφή:
MyOpenWeatherMapApiKey=”YOUR_API_KEY”

2. Στον κωδικά σας, αλλάξτε το αρχείο app/build.gradle 

android {

....
....
...
    buildTypes.each {
        it.buildConfigField 'String', 'OPEN_WEATHER_MAP_API_KEY', MyOpenWeatherMapApiKey
    }
}
```


As mentioned in the previous node, we’re going to be adding a CursorAdapter to Sunshine called ForecastAdapter. Why? Well, our loader will be making the calls on our content provider to get a Cursorand we’ll want to take the data from that cursor and put it into our UI. This is what adapters are meant for, associating data with UI components.

For now, we’re not going to worry about the Loader and we're just going to change our code to use a more appropriate adapter. Right now, you might recall from lesson 1, we’re using an ArrayAdapter. This ArrayAdapter is populated only when we’re syncing with OpenWeatherMapAPI. Basically we get the JSON, put it in the content provider, take it out again, and change it to an array. This is not ideal. Let’s fix all of this.

We’ll talk more about making an adapter in Lesson 5, for now, we’re going to give you the code for one and do a quick overview.
Go ahead and copy ForecastAdapter and Utility.java into the main package (example.android.com) of your code.

## What’s going inside ForecastAdapter

ForecastAdapter is a subclass of CursorAdapter. Let’s take a quick look at the functionality we’ve got in here. After the constructor, there are four methods. The first two are formatHighLows and convertCursorRowToUXFormat. These are two formatting methods specific to Sunshine.

- convertCursorRowToUXFormat takes a row from a cursor and constructs a single string of the format: Date - Weather -- High/Low
This is the string we’re used to seeing in the listview element. It uses formatHighLow to get the correct string for the temperature.
The other two methods are necessary to override whenever you’re extending a cursor adapter.

- newView - Remember that adapters work with listviews to populate them. They create duplicates of the same layout to put into the list view. This is where you return what layout is going to be duplicated.

- View view = LayoutInflater.from(context).inflate(R.layout.list_item_forecast, parent, false);
- return view;

In our case, we’re inflating our listview layout, list_item_forecast, and then returning it.

- bindView - This is where the exciting bit occurs. As the name suggests you are binding the values in the cursor to the view.

- TextView tv = (TextView)view;

- tv.setText(convertCursorRowToUXFormat(cursor));

The View passed into bindView is the View returned from newView. We know it’s a TextView, so we cast it. Then we take the Cursor, run it through our custom made formatting function, and set the text of the TextView.


## Refactoring to use ForecastAdapter with Cursors and from the Fragment

1. Change mForecastAdapter's type
Change mForecastAdapter, to be an instance of ForecastAdapter.

2. Get Data from the Database
Let’s go to where we first need to populate the ForecastFragment with data and do so by getting the data from the database. Go to onCreateView. Use WeatherProvider to query the database the same way you are in FetchWeatherTask:
```
     String locationSetting = Utility.getPreferredLocation(getActivity());
        // Sort order:  Ascending, by date.
        String sortOrder = WeatherContract.WeatherEntry.COLUMN_DATE + " ASC";
        Uri weatherForLocationUri = WeatherContract.WeatherEntry.buildWeatherLocationWithStartDate(
                locationSetting, System.currentTimeMillis());

        Cursor cur = getActivity().getContentResolver().query(weatherForLocationUri,
                null, null, null, sortOrder);
```
                
3. Make a new ForecastAdapter

Still in onCreateView, we have a Cursor cur, so let’s use our new ForecastAdapter. Create a new ForecastAdapter with the new cursor. The list will be empty the first time we run.

mForecastAdapter = new ForecastAdapter(getActivity(), cur, 0);

4. Delete OnItemClickListener

Because we changed the adapter, the OnItemClickListener in ForecastFragment for the ListView won’t work. 
Specifically this line 
``` String forecast = mForecastAdapter.getItem(position);```
is problematic because getItem with a CursorAdapter doesn’t return a string.

Go ahead and remove or comment out this for now.

We’ll talk more about this and correct this soon enough. Until then, our code will compile and run but not have access to our DetailView.

5. Clean up

Inside of FetchWeatherTask, we’re going to remove the formatting code and anything for updating the adapter. _You can remove:_

- Any reference to mForecastAdapter

- getReadableDateString, formatHighLows, convertContentValuesToUXFormat. These are all formatting functions and we’ve moved them to the ForecastAdapter.

- The lines in getWeatherDataFromJson where we requery the database after the insert.

- PostExecute

Note: To keep your tests working, you'll need to modify line 42 of TestFetchWeatherTask to be:

``` FetchWeatherTask fwt = new FetchWeatherTask(getContext()); ```




