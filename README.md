# PRP
Perfect - RIde - Passenger
#Postalcode Validation- code
public void afterTextChanged(Editable editable) {
            String name = editable.toString();
            if (name.length() == 6) {
                boolean charCheck = true;
                boolean digitCheck = true;
                char[] chars = new char[6];
                name.getChars(0, name.length() - 1, chars, 0);
                for (int i = 0; i < name.length(); i++) {
                    if (i % 2 == 0) {
                        if (Character.isDigit(chars[i])) {
                            charCheck = false;
                        }
                    } else {
                        if (Character.isLetter(chars[i])) {
                            digitCheck = false;
                        }
                    }
                }
                if (charCheck && digitCheck) {
                    rootView.findViewById(R.id.search_btn).setVisibility(View.VISIBLE);
                } else {
                    new AlertDialog.Builder(getActivity())
                            .setTitle("PerfectRide")
                            .setMessage("Not a valid postal code.")
                            .setCancelable(false)
                            .setPositiveButton("ok", new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialogInterface, int i) {
                                }
                            }).show();
                }
