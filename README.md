@SuppressWarnings("rawtypes")
	public String validateDownloadFile() {
		String home = "";
		String fileName = "";
		String currentHandle = driver.getCurrentUrl();
		try {
			Thread.sleep(15000);
			int count = 1;
			do {
				if (get_downloaded_files().toString().contains(".xlsx")
						|| get_downloaded_files().toString().contains(".csv")) {
					logger.info("FILE DOWNLOADED TO GRID NODE");
					break;
				} else {
					logger.info("DOWNLOAD PROGRESS: " + get_download_progress_all());
				}
				count++;
				Thread.sleep(5000);
			} while (count < 11);
			ArrayList downloaded_files_arraylist = get_downloaded_files();
			String content = get_file_content(downloaded_files_arraylist.get(0).toString());
			home = System.getProperty("user.home") + "/Downloads";
			FileUtils.createDirectory(home);
			fileName = downloaded_files_arraylist.get(0).toString().split("Downloads/")[1];
			FileOutputStream fos = new FileOutputStream(home + "//" + fileName);
			byte[] decoder = Base64.decodeBase64(content.substring(content.indexOf("base64,") + 7));
			fos.write(decoder);
			logger.info("File saved to local =>>" + home + "//" + fileName);
			driver.get(currentHandle);
			fos.close();
		} catch (Exception e) {
			e.printStackTrace();
			driver.get(currentHandle);
		}
		return home + "//" + fileName;
	}

	private String get_file_content(String path) {
		String file_content = null;
		try {
			if (!driver.getCurrentUrl().startsWith("chrome://downloads")) {
				// driver.findElement(By.cssSelector("body")).sendKeys(Keys.CONTROL + "t");
				driver.get("chrome://downloads/");
			}
			WebElement elem = (WebElement) jsExecutor.executeScript(
					"var input = window.document.createElement('INPUT'); " + "input.setAttribute('type', 'file'); "
							+ "input.hidden = true; " + "input.onchange = function (e) { e.stopPropagation() }; "
							+ "return window.document.documentElement.appendChild(input); ",
					"");
			elem.sendKeys(path);
			file_content = (String) jsExecutor.executeAsyncScript("var input = arguments[0], callback = arguments[1]; "
					+ "var reader = new FileReader(); " + "reader.onload = function (ev) { callback(reader.result) }; "
					+ "reader.onerror = function (ex) { callback(ex.message) }; "
					+ "reader.readAsDataURL(input.files[0]); " + "input.remove(); ", elem);
			if (!file_content.startsWith("data:")) {
				logger.info("Failed to get file content");
			}

		} catch (Exception e) {
			logger.info(e + "");
		}
		return file_content;

	}

	@SuppressWarnings("rawtypes")
	private ArrayList get_downloaded_files() {
		ArrayList filesFound = null;
		try {
			if (!driver.getCurrentUrl().equals("chrome://downloads")) {
				driver.get("chrome://downloads/");
			}
			filesFound = (ArrayList) jsExecutor.executeScript("return  document.querySelector('downloads-manager')  "
					+ " .shadowRoot.querySelector('#downloadsList')         "
					+ " .items.filter(e => e.state === 'COMPLETE')          "
					+ " .map(e => e.filePath || e.file_path || e.fileUrl || e.file_url); ", "");
		} catch (Exception e) {
			logger.info(e + "");
		}
		return filesFound;
	}

	@SuppressWarnings("unused")
	private String get_download_progress() {
		String progress = null;
		try {
			if (!driver.getCurrentUrl().startsWith("chrome://downloads")) {
				// driver.findElement(By.cssSelector("body")).sendKeys(Keys.CONTROL + "t");
				driver.get("chrome://downloads/");
			}
			progress = (String) jsExecutor
					.executeScript("var tag = document.querySelector('downloads-manager').shadowRoot;"
							+ "var intag = tag.querySelector('downloads-item').shadowRoot;"
							+ "var progress_tag = intag.getElementById('progress');" + "var progress = null;"
							+ " if(progress_tag) { " + "    progress = progress_tag.value; " + "  }"
							+ "return progress;", "");

		} catch (Exception e) {
			logger.info(e + "");
		}
		return progress;
	}

	@SuppressWarnings("rawtypes")
	private ArrayList get_download_progress_all() {
		ArrayList progress = null;
		try {
			if (!driver.getCurrentUrl().startsWith("chrome://downloads")) {
				driver.get("chrome://downloads/");
			}
			progress = (ArrayList) jsExecutor
					.executeScript(" var tag = document.querySelector('downloads-manager').shadowRoot;"
							+ "			    var item_tags = tag.querySelectorAll('downloads-item');"
							+ "			    var item_tags_length = item_tags.length;"
							+ "			    var progress_lst = [];"
							+ "			    for(var i=0; i<item_tags_length; i++) {"
							+ "			        var intag = item_tags[i].shadowRoot;"
							+ "			        var progress_tag = intag.getElementById('progress');"
							+ "			        var progress = null;" + "			        if(progress_tag) {"
							+ "			            var progress = progress_tag.value;" + "			        }"
							+ "			        progress_lst.push(progress);" + "			    }"
							+ "			    return progress_lst", "");

		} catch (Exception e) {
			logger.info(e + "");
		}
		return progress;
	}
