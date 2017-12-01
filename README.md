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




# Moving to Cursor Adapter


As mentioned in the previous node, we’re going to be adding a CursorAdapter to Sunshine called ForecastAdapter. Why? Well, our loader will be making the calls on our content provider to get a Cursorand we’ll want to take the data from that cursor and put it into our UI. This is what adapters are meant for, associating data with UI components.

For now, we’re not going to worry about the Loader and we're just going to change our code to use a more appropriate adapter. Right now, you might recall from lesson 1, we’re using an ArrayAdapter. This ArrayAdapter is populated only when we’re syncing with OpenWeatherMapAPI. Basically we get the JSON, put it in the content provider, take it out again, and change it to an array. This is not ideal. Let’s fix all of this.



### Go ahead and copy [ForecastAdapter](https://github.com/udacity/Sunshine-Version-2/blob/4.18_cursor_adapter/app/src/main/java/com/example/android/sunshine/app/ForecastAdapter.java) and [Utility.java](https://github.com/udacity/Sunshine-Version-2/blob/4.18_cursor_adapter/app/src/main/java/com/example/android/sunshine/app/Utility.java) into the main package (example.android.com) of your code.

## Forecast Adapter

```
package com.example.android.sunshine.app;

import android.content.Context;
import android.database.Cursor;
import android.support.v4.widget.CursorAdapter;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import com.example.android.sunshine.app.data.WeatherContract;

/**
 * {@link ForecastAdapter} exposes a list of weather forecasts
 * from a {@link android.database.Cursor} to a {@link android.widget.ListView}.
 */
public class ForecastAdapter extends CursorAdapter {
    public ForecastAdapter(Context context, Cursor c, int flags) {
        super(context, c, flags);
    }

    /**
     * Prepare the weather high/lows for presentation.
     */
    private String formatHighLows(double high, double low) {
        boolean isMetric = Utility.isMetric(mContext);
        String highLowStr = Utility.formatTemperature(high, isMetric) + "/" + Utility.formatTemperature(low, isMetric);
        return highLowStr;
    }

    /*
        This is ported from FetchWeatherTask --- but now we go straight from the cursor to the
        string.
     */
    private String convertCursorRowToUXFormat(Cursor cursor) {
        // get row indices for our cursor
        int idx_max_temp = cursor.getColumnIndex(WeatherContract.WeatherEntry.COLUMN_MAX_TEMP);
        int idx_min_temp = cursor.getColumnIndex(WeatherContract.WeatherEntry.COLUMN_MIN_TEMP);
        int idx_date = cursor.getColumnIndex(WeatherContract.WeatherEntry.COLUMN_DATE);
        int idx_short_desc = cursor.getColumnIndex(WeatherContract.WeatherEntry.COLUMN_SHORT_DESC);

        String highAndLow = formatHighLows(
                cursor.getDouble(idx_max_temp),
                cursor.getDouble(idx_min_temp));

        return Utility.formatDate(cursor.getLong(idx_date)) +
                " - " + cursor.getString(idx_short_desc) +
                " - " + highAndLow;
    }

    /*
        Remember that these views are reused as needed.
     */
    @Override
    public View newView(Context context, Cursor cursor, ViewGroup parent) {
        View view = LayoutInflater.from(context).inflate(R.layout.list_item_forecast, parent, false);

        return view;
    }

    /*
        This is where we fill-in the views with the contents of the cursor.
     */
    @Override
    public void bindView(View view, Context context, Cursor cursor) {
        // our view is pretty simple here --- just a text view
        // we'll keep the UI functional with a simple (and slow!) binding.

        TextView tv = (TextView)view;
        tv.setText(convertCursorRowToUXFormat(cursor));
    }
}
```

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



# Use of Loaders in Forecast Fragment


```
public class ForecastFragment extends Fragment implements LoaderManager.LoaderCallbacks<Cursor> {
    private static final int FORECAST_LOADER = 0;
    private ForecastAdapter mForecastAdapter;
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        getLoaderManager().initLoader(FORECAST_LOADER, null, this);
        super.onActivityCreated(savedInstanceState);
    }

    private void updateWeather() {
        FetchWeatherTask weatherTask = new FetchWeatherTask(getActivity());
        String location = Utility.getPreferredLocation(getActivity());
        weatherTask.execute(location);
    }

    @Override
    public void onStart() {
        super.onStart();
        updateWeather();
    }

    @Override
    public Loader<Cursor> onCreateLoader(int i, Bundle bundle) {
        String locationSetting = Utility.getPreferredLocation(getActivity());

        // Sort order:  Ascending, by date.
        String sortOrder = WeatherContract.WeatherEntry.COLUMN_DATE + " ASC";
        Uri weatherForLocationUri = WeatherContract.WeatherEntry.buildWeatherLocationWithStartDate(
                locationSetting, System.currentTimeMillis());
        return new CursorLoader(getActivity(),
                weatherForLocationUri,  null, null, null, sortOrder);
    }

    @Override
    public void onLoadFinished(Loader<Cursor> cursorLoader, Cursor cursor) {
        mForecastAdapter.swapCursor(cursor);
    }

    @Override
    public void onLoaderReset(Loader<Cursor> cursorLoader) {
        mForecastAdapter.swapCursor(null);
    }
 
```
 
# Projections

```
private static final String[] FORECAST_COLUMNS = {
            // In this case the id needs to be fully qualified with a table name, since
            // the content provider joins the location & weather tables in the background
            // (both have an _id column)
            // On the one hand, that's annoying.  On the other, you can search the weather table
            // using the location set by the user, which is only in the Location table.
            // So the convenience is worth it.
            WeatherContract.WeatherEntry.TABLE_NAME + "." + WeatherContract.WeatherEntry._ID,
            WeatherContract.WeatherEntry.COLUMN_DATE,
            WeatherContract.WeatherEntry.COLUMN_SHORT_DESC,
            WeatherContract.WeatherEntry.COLUMN_MAX_TEMP,
            WeatherContract.WeatherEntry.COLUMN_MIN_TEMP,
            WeatherContract.LocationEntry.COLUMN_LOCATION_SETTING,
            WeatherContract.WeatherEntry.COLUMN_WEATHER_ID,
            WeatherContract.LocationEntry.COLUMN_COORD_LAT,
            WeatherContract.LocationEntry.COLUMN_COORD_LONG
    };

    // These indices are tied to FORECAST_COLUMNS.  If FORECAST_COLUMNS changes, these
    // must change.
    static final int COL_WEATHER_ID = 0;
    static final int COL_WEATHER_DATE = 1;
    static final int COL_WEATHER_DESC = 2;
    static final int COL_WEATHER_MAX_TEMP = 3;
    static final int COL_WEATHER_MIN_TEMP = 4;
    static final int COL_LOCATION_SETTING = 5;
    static final int COL_WEATHER_CONDITION_ID = 6;
    static final int COL_COORD_LAT = 7;
    static final int COL_COORD_LONG = 8;


private String convertCursorRowToUXFormat(Cursor cursor) {
        String highAndLow = formatHighLows(
                cursor.getDouble(ForecastFragment.COL_WEATHER_MAX_TEMP),
                cursor.getDouble(ForecastFragment.COL_WEATHER_MIN_TEMP));

        return Utility.formatDate(cursor.getLong(ForecastFragment.COL_WEATHER_DATE)) +
                " - " + cursor.getString(ForecastFragment.COL_WEATHER_DESC) +
                " - " + highAndLow;
    }

```

 
# Make Details View Functional


One of the things that we decided to temporarily break is the details view. 
It’s time to fix this and hook things up.
The major change we will make here is that we will start our DetailsActivity by 
passing it the URI it needs to pass to the content provider to get the correct data.

1. Add OnItemClickListener to ListView

In ForecastFragment, in the onCreateView method, go ahead and add an ```onItemClickListener```, 
except this time, it’s going to pass a URI for the data needed for the detail view.

```
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {

            @Override
            public void onItemClick(AdapterView adapterView, View view, int position, long l) {
                // CursorAdapter returns a cursor at the correct position for getItem(), or null
                // if it cannot seek to that position.
                Cursor cursor = (Cursor) adapterView.getItemAtPosition(position);
                if (cursor != null) {
                    String locationSetting = Utility.getPreferredLocation(getActivity());
                    Intent intent = new Intent(getActivity(), DetailActivity.class)
                            .setData(WeatherContract.WeatherEntry.buildWeatherLocationWithDate(
                                    locationSetting, cursor.getLong(COL_WEATHER_DATE)
                            ));
                    startActivity(intent);
                }
            }
        });
```

2. Print the URI in the ListView

On the ```DetailActivity``` side, we'll want to change the code, which is referring to an 
intent extra that you're no longer setting. Instead we used setData so we need to grab this data using ```getDataString```. 

The full code you'll need to put in DetailActivity is:

```
       if (intent != null) {
           mForecastStr = intent.getDataString();
       }
```

This causes the detail view to show the URI.

But wait, that's not what we want! So obviously we’re not done at this point, 
we need to actually use the URI to display the correct data in the detail view. 
You’ll be doing that in the next node.

Check out the full diff [here](https://github.com/udacity/Sunshine-Version-2/compare/4.20_projections...4.21_details_view)
 
 
# Implement Details View as Cursor Loader (+use Projections)

```

public static class DetailFragment extends Fragment implements LoaderCallbacks<Cursor> {
        private static final String LOG_TAG = DetailFragment.class.getSimpleName();
        private static final String FORECAST_SHARE_HASHTAG = " #SunshineApp";
        private ShareActionProvider mShareActionProvider;
        private String mForecast;
        private static final int DETAIL_LOADER = 0;


        private static final String[] FORECAST_COLUMNS = {
                WeatherEntry.TABLE_NAME + "." + WeatherEntry._ID,
                WeatherEntry.COLUMN_DATE,
                WeatherEntry.COLUMN_SHORT_DESC,
                WeatherEntry.COLUMN_MAX_TEMP,
                WeatherEntry.COLUMN_MIN_TEMP,
        };
        // these constants correspond to the projection defined above, and must change if the
        // projection changes
        private static final int COL_WEATHER_ID = 0;
        private static final int COL_WEATHER_DATE = 1;
        private static final int COL_WEATHER_DESC = 2;
        private static final int COL_WEATHER_MAX_TEMP = 3;
        private static final int COL_WEATHER_MIN_TEMP = 4;


        public DetailFragment() {
            setHasOptionsMenu(true);
        }
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                 Bundle savedInstanceState) {
            return inflater.inflate(R.layout.fragment_detail, container, false);
        }
        @Override
        public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
            // Inflate the menu; this adds items to the action bar if it is present.
            inflater.inflate(R.menu.detailfragment, menu);
            // Retrieve the share menu item
            MenuItem menuItem = menu.findItem(R.id.action_share);


            // Get the provider and hold onto it to set/change the share intent.
            mShareActionProvider = (ShareActionProvider) MenuItemCompat.getActionProvider(menuItem);
            // If onLoadFinished happens before this, we can go ahead and set the share intent now.
            if (mForecast != null) {
                mShareActionProvider.setShareIntent(createShareForecastIntent());
            }
        }
        private Intent createShareForecastIntent() {
            Intent shareIntent = new Intent(Intent.ACTION_SEND);
            shareIntent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET);
            shareIntent.setType("text/plain");
            shareIntent.putExtra(Intent.EXTRA_TEXT, mForecast + FORECAST_SHARE_HASHTAG);
            return shareIntent;
        }
        @Override
        public void onActivityCreated(Bundle savedInstanceState) {
            getLoaderManager().initLoader(DETAIL_LOADER, null, this);
            super.onActivityCreated(savedInstanceState);
        }
        @Override
        public Loader<Cursor> onCreateLoader(int id, Bundle args) {
            Log.v(LOG_TAG, "In onCreateLoader");
            Intent intent = getActivity().getIntent();
            if (intent == null) {
                return null;
            }
            // Now create and return a CursorLoader that will take care of
            // creating a Cursor for the data being displayed.
            return new CursorLoader(
                    getActivity(),
                    intent.getData(),
                    FORECAST_COLUMNS, null,   null,      null);

            );
        }
        @Override
        public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
            Log.v(LOG_TAG, "In onLoadFinished");
            if (!data.moveToFirst()) { return; }
            String dateString = Utility.formatDate(
                    data.getLong(COL_WEATHER_DATE));
            String weatherDescription =
                    data.getString(COL_WEATHER_DESC);
            boolean isMetric = Utility.isMetric(getActivity());
            String high = Utility.formatTemperature(
                    data.getDouble(COL_WEATHER_MAX_TEMP), isMetric);
            String low = Utility.formatTemperature(
                    data.getDouble(COL_WEATHER_MIN_TEMP), isMetric);
            mForecast = String.format("%s - %s - %s/%s", dateString, weatherDescription, high, low);
            TextView detailTextView = (TextView)getView().findViewById(R.id.detail_text);
            detailTextView.setText(mForecast);
            // If onCreateOptionsMenu has already happened, we need to update the share intent now.
            if (mShareActionProvider != null) {
                mShareActionProvider.setShareIntent(createShareForecastIntent());
            }
        }
        @Override
        public void onLoaderReset(Loader<Cursor> loader) { }
    }
```


