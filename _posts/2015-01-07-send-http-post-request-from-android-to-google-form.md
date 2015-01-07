---
layout: post
title: send HTTP POST request from Android to Google Form
date: 2015-01-07 23:15:00 +08:00
description: ""
headline: ""
categories: android
tags: 
  - Android
  - Programming
comments: true
mathjax: null
featured: false
share: true
published: true
---

When you want to collect data from your app, if the data is for temporary use, passing data to Google forms may be a better option than implementing your own server-side APIs. This article shows how to send HTTP POST request from Android application to Google form.

First of all, create a form.

Go to [Google Forms](https://www.google.com/forms/about/) and click "Create a free form".

Then add items to your form. You can take the form as a table in a database: every request you send is a row (record) and each question is a column.

You can get the form ID (which is "1rJGPa_xJKugj5gDlm98CIT5khIEvHUOagmDMaykGzAM" in image below) from browser's address bar. After done editing, click "send form" at the bottom of the page to save your form.

<img border="0" src="/images/20150107/img_form_edit.png" width="800"/>

Click "View response" button on the top menu to open the response sheet; this is where you save your data.

Then click "View live form" button on the top menu (right to the "view response" button) to open the form.

<img border="0" src="/images/20150107/img4.png" width="600" />

At this page, right-click on the page and choose "View Page Source".

Let's work on question 1 first. Search for "Question 1" (your question name) and copy the name attribute "entry.1075601256". This is what we use to send with our request.

<img border="0" src="/images/20150107/img_q1.png"/>

For question 2, which is a checkboxes question in this example, get the name attribute and value attribute for each checkbox item.

<img border="0" src="/images/20150107/img_q2.png"/>

OK, now you've got all the things you need! It's time to go back to your Android project.

In your Android project, get the data you want and send them with HTTP POST methods.

    public class Main extends Activity {

        final String GOOGLE_FORM_ID = "1rJGPa_xJKugj5gDlm98CIT5khIEvHUOagmDMaykGzAM";
        final String strUrl = "https://docs.google.com/forms/d/" + GOOGLE_FORM_ID + "/formResponse";

        final String q1Entry = "entry.1075601256";
        final String q2Entry = "entry.1904375493";
        final String q2Value[] = { "Option 1", "Option 2", "Option 3" };

        LinearLayout ll;
        EditText et;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.layout_main);

            AlertDialog.Builder alert = new AlertDialog.Builder(this);
            alert.setTitle("DIALOG TITLE");

            ll = new LinearLayout(this);
            ll.setOrientation(LinearLayout.VERTICAL);
            et = new EditText(this);
            ll.addView(et);

            for (int i = 0; i < q2Value.length; i++) {
                CheckBox cb = new CheckBox(this);
                cb.setText(q2Value[i]);
                ll.addView(cb);
            }

            alert.setView(ll);

            alert.setPositiveButton("Send", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int id) {
                    new Thread(new Runnable() {
                        public void run() {
                            try {
                                URL url = new URL(strUrl);
                                HttpsURLConnection conn = (HttpsURLConnection) url.openConnection();
                                conn.setRequestMethod("POST");

                                List<NameValuePair> params = new ArrayList<NameValuePair>();
                                for (int i = 0; i < ll.getChildCount(); i++) {
                                    View v = ll.getChildAt(i);
                                    if (v instanceof EditText)
                                        params.add(new BasicNameValuePair(q1Entry,
                                                ((EditText) v).getText().toString()));
                                    else if (((CheckBox) v).isChecked())
                                        params.add(new BasicNameValuePair(q2Entry,
                                                ((CheckBox) v).getText().toString()));
                                }
                                UrlEncodedFormEntity entity = new UrlEncodedFormEntity(params);
                                OutputStream post = conn.getOutputStream();
                                entity.writeTo(post);
                                post.flush();

                                BufferedReader in = new BufferedReader(
                                                            new InputStreamReader(conn.getInputStream()));
                                String inputLine, response = "";
                                while ((inputLine = in.readLine()) != null) {
                                    response += inputLine;
                                }
                                Log.d("Your app", response);
                                post.close();
                                in.close();
                            } catch (Exception e) {
                                Log.e("Your app", "error", e);
                            }
                        }
                    }).start();
                }
            });
            alert.show();
        }
    }

Here's what the activity looks like:

<img border="0" src="/images/20150107/img_mobile.png" width="400"/>

After click send button, the sent data will immediately show up in the response sheet.

<img border="0" src="/images/20150107/img_response.png"/>

That's it!

