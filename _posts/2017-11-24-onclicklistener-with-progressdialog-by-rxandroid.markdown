---
layout: post
title: "OnClickListener with ProgressDialog by RxAndroid"
date: "2017-11-24 17:52"
author: 'u-ryo'
categories: [android, RxAndroid, RxJava, ProgressDialog]
comments: true
published: true
---
「clickしたらbackgroundで処理して
その間ProgressDialog出して
終わったらProgressDialog消して
終了/失敗dialogを表示する」のを
RxAndroid(AxJava)でやる、
というのは、
`using()`を使うといいらしいです。
cf. [RxJavaを使った通信中にProgressダイアログを出す](https://qiita.com/boohbah/items/e8010730725c54f85a3a)

元々がretrofit2を使ってないので、
retrofit2を使うともうちょっと違うかも。

```
@Override
public void onClick(View v) {
    LogUtil.d("診断取得結果をuploadする button");
    uploadButtonEnable(false);
    if (!activity.networkCheck()) {
        activity.genAlertDialog(activity.getString(
                R.string.no_network_connectivity_available_message),
                (dialog, which) -> {});
        uploadButtonEnable(true);
        return;
    }
    Single.using(this::showProgressDialog,
            dialog -> Single.<Boolean>create(this::setUploadSubscriber)
                    .subscribeOn(Schedulers.newThread())
                    .observeOn(AndroidSchedulers.mainThread()),
            Dialog::dismiss)
            .subscribe(this::controlUploadButtonWithDialog,
                    this::showUploadFailureDialog);
}

private ProgressDialog showProgressDialog() {
    ProgressDialog dialog = new ProgressDialog(activity);
    dialog.setMessage(activity.getString(R.string.history_uploading));
    dialog.setProgressStyle(ProgressDialog.STYLE_SPINNER);
    dialog.show();
    LogUtil.d(dialog.toString());
    return dialog;
}

private void setUploadSubscriber(SingleSubscriber<? super Boolean> subscriber) {
    View historyListView = activity.findViewById(R.id.historyListView);
    List<String> selectedList = new ArrayList<>();
    Adapter adapter = null;
    if (historyListView != null) {
        adapter = ((ListView) historyListView).getAdapter();
        for (int i = 0; i < adapter.getCount(); i++) {
            Map<String, String> historyItems = (Map<String, String>) adapter.getItem(i);
            if ("true".equals(historyItems.get("historySelected"))) {
                selectedList.add(historyItems.get("historyCatalogID"));
            }
        }

        if (selectedList.isEmpty()) {
            subscriber.onSuccess(false);
            return;
        }
    }

    try {
        ProcessUtil.callReportDataAll(commonBean.toMapFull(), activity, selectedList);
        if ((!selectedList.isEmpty()
                && !ProcessUtil.uploadSucceeded(selectedList, activity))
                || (selectedList.isEmpty()
                && !ProcessUtil.lastUploadSucceeded(activity))) {
            subscriber.onError(new RuntimeException(""));
            return;
        }
        ProcessUtil.sendTerminalUsageHistory(commonBean.toMap(), activity);
        subscriber.onSuccess(true);
        LogUtil.d(selectedList.toString());

        if (adapter != null) {
            for (int i = 0; i < adapter.getCount(); i++) {
                Map<String, String> historyItems = (Map<String, String>) adapter.getItem(i);
                if ("true".equals(historyItems.get("historySelected"))) {
                    historyItems.put("historySelected", "false");
                    historyItems.put("historySaved",
                            activity.getString(R.string.history_list_already_saved));
                }
            }
        }
    } catch (Exception e) {
        LogUtil.e(e);
        subscriber.onError(e);
    }
}

private void controlUploadButtonWithDialog(boolean hasItemSelected) {
    ListView listView = (ListView) activity.findViewById(R.id.historyListView);
    if (hasItemSelected) {
        activity.genAlertDialog(activity.getString(
                R.string.diagnosis_result_upload_success_message),
                (dialog, which) -> {});
        if (listView != null) {
            for (int i = 0; i < listView.getChildCount(); i++) {
                CheckBox checkBox = (CheckBox) listView.getChildAt(i)
                        .findViewById(R.id.historySelected);
                if (checkBox.isChecked()) {
                    checkBox.setChecked(false);
                    checkBox.setEnabled(false);
                    TextView saved = (TextView) listView.getChildAt(i)
                            .findViewById(R.id.historySaved);
                    saved.setText(activity.getString(R.string.history_list_already_saved));
                    saved.setTextColor(Color.GRAY);
                }
            }
        }
    } else {
        activity.genAlertDialog(activity.getString(R.string.history_nothing_checked),
                (dialog, which) -> {});
    }
    uploadButtonEnable(false);
}

private void showUploadFailureDialog(Throwable e) {
    uploadButtonEnable(true);
    LogUtil.e(checkStr(e.getMessage()), e);
    activity.genAlertDialog(activity.getString(
            R.string.diagnosis_result_upload_failure_message)
                    + "\n" + e.getMessage(),
            (dialog, which) -> {});
}

public void uploadButtonEnable(boolean enable) {
    uploadButton.setEnabled(enable);
    if (enable) {
        uploadButton.getBackground().setColorFilter(null);
    } else {
        uploadButton.getBackground()
                .setColorFilter(Color.GRAY, PorterDuff.Mode.MULTIPLY);
    }
}
```
