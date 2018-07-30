# PRP
Perfect - RIde - Passenger
Distance and Fare Calculation 
public class RideDetailsFragment extends Fragment {

    private View rootView;
    private EditText destinationET;
    private EditText distanceEt;
    private EditText estimateEt;
    private String location;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        rootView = inflater.inflate(R.layout.fragment_ride_details, container, false);
        destinationET = rootView.findViewById(R.id.destination_et);
        distanceEt = rootView.findViewById(R.id.distance_et);
        estimateEt = rootView.findViewById(R.id.estimate_et);
        Button map_button = rootView.findViewById(R.id.map_button);
        Button confirm_button = rootView.findViewById(R.id.confirm_button);
        location = getArguments().getString("AIzaSyD_rBV3vktFJqIWqp8nkH1K-8pc-2GEslo", ""); /* calculate the estimate cad */
        if (location == null || location.isEmpty()) {
            location = "43.780257, -79.232250";  /* longitude and latitude*/
        }
        map_button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent(getActivity(), ActivityMap.class);
                intent.putExtra("startLocation", location);
                intent.putExtra("endLocation", getArguments().getString("postalCode"));
                startActivity(intent);
            }
        });

        confirm_button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(getActivity(), "Success", Toast.LENGTH_SHORT).show();
                getActivity().onBackPressed();
            }
        });
        destinationET.setText("");
        distanceEt.setText("km"); /*remove the set text km */
        estimateEt.setText("");

        new getDistanceFromAPI().execute();

        return rootView;

    }

    private class getDistanceFromAPI extends AsyncTask<Void, Void, String> {

        @Override
        protected String doInBackground(Void... params) {
            HttpURLConnection urlConnection = null;
            BufferedReader reader = null;

            String forecastJsonStr = null;

            try {
                URL url = new URL("http://maps.googleapis.com/maps/api/distancematrix/json?origins=" + location + "&destinations=" + getArguments().getString("postalCode") + "&mode=driving&language=en-EN&sensor=false");

                urlConnection = (HttpURLConnection) url.openConnection();
                urlConnection.setRequestMethod("GET");
                urlConnection.connect();

                InputStream inputStream = urlConnection.getInputStream();
                StringBuffer buffer = new StringBuffer();
                if (inputStream == null) {
                    return null;
                }
                reader = new BufferedReader(new InputStreamReader(inputStream));

                String line;
                while ((line = reader.readLine()) != null) {
                    buffer.append(line + "\n");
                }

                if (buffer.length() == 0) {
                    return null;
                }
                forecastJsonStr = buffer.toString();
                return forecastJsonStr;
            } catch (IOException e) {
                Log.e("Fragment", "Error ", e);
                return null;
            } finally {
                if (urlConnection != null) {
                    urlConnection.disconnect();
                }
                if (reader != null) {
                    try {
                        reader.close();
                    } catch (final IOException e) {
                        Log.e("Fragment", "Error closing stream", e);
                    }
                }
            }
        }

        @Override
        protected void onPostExecute(String s) {
            super.onPostExecute(s);
            try {
                JSONObject object = new JSONObject(s);
                if (object.optString("status").equalsIgnoreCase("OK")) {
                    String destination_addresses = object.optString("destination_addresses");
                    String substring = destination_addresses.substring(2, destination_addresses.length() - 2);
                    if (substring == null || substring.isEmpty()) {
                        Toast.makeText(getActivity(), "There is no location for this postal code", Toast.LENGTH_SHORT).show();
                        getActivity().onBackPressed();
                    } else {
                        destinationET.setText(substring);
                        JSONObject rows = (JSONObject) object.getJSONArray("rows").get(0);
                        JSONObject elements = (JSONObject) rows.getJSONArray("elements").get(0);
                        JSONObject distance = elements.getJSONObject("distance");
                        int text = distance.optInt("value");
                        double ceil = Math.ceil((double) text / 1000);
                        distanceEt.setText(ceil + " km");
                        estimateEt.setText(ceil + "");
                    }

                } else {

                }
            } catch (JSONException e) {
                e.printStackTrace();
            }
        Log.i("json", s);
        }
    }
}

Layout - XML

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">


    <LinearLayout
        android:id="@+id/profile_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/profile_photo"
        android:layout_marginBottom="@dimen/activity_vertical_margin"
        android:layout_marginTop="@dimen/activity_vertical_margin"
        android:orientation="horizontal"
        android:weightSum="3">

        <LinearLayout
            android:id="@+id/fname_layout"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical"
            android:weightSum="3">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="@dimen/activity_vertical_margin"
                android:layout_weight="1"
                android:gravity="center"
                android:text="Destination" />


            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="25dp"
                android:layout_weight="1"
                android:gravity="center"
                android:text="Distance" />

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="30dp"
                android:layout_weight="1"
                android:gravity="center"
                android:text="Estimate CAD$" />


        </LinearLayout>

        <LinearLayout
            android:id="@+id/lname_layout"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="2"
            android:orientation="vertical"
            android:weightSum="3">

            <EditText
                android:id="@+id/destination_et"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:textColor="#000"
                android:enabled="false"
                android:imeOptions="actionNext" />

            <EditText
                android:id="@+id/distance_et"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:enabled="false"
                android:textColor="#000"
                android:imeOptions="actionDone" />

            <EditText
                android:id="@+id/estimate_et"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:enabled="false"
                android:textColor="#000"
                android:imeOptions="actionDone" />
        </LinearLayout>
    </LinearLayout>

    <Button
        android:id="@+id/map_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/profile_layout"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="@dimen/activity_vertical_margin"
        android:text="Map" />

    <Button
        android:id="@+id/confirm_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/map_button"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="@dimen/activity_vertical_margin"
        android:text="Confirm" />

</RelativeLayout>



