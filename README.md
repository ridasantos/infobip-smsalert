# infobip-smsalert 
Basic messaging example using infobip in c#

Reference the project OneApi or get the DLL from the project.

Initialize SMSClient with username and password

    OneApi.Config.Configuration config = new OneApi.Config.Configuration(username, password);
    SMSClient smsClient = new SMSClient(config);

# Sending message with multiple address
Note: for single destination address, pass a string instead of string[]

    OneApi.Config.Configuration config = new OneApi.Config.Configuration(username, password);
    SMSClient smsClient = new SMSClient(config);

    SMSRequest smsRequest = new SMSRequest("username", message, destinationAddress);
    smsRequest.Language = new OneApi.Language(OneApi.LanguageCode.Default);

    var result = smsClient.SmsMessagingClient.SendSMS(smsRequest);
    
 # Sending scheduled message
 
     string username = ConfigurationManager.AppSettings["Username"].ToString(),
         password = ConfigurationManager.AppSettings["Password"].ToString();

     string encryptedCredentials = EncryptCredential(username, password);

     var httpWebRequest = (HttpWebRequest)WebRequest.Create("http://api.infobip.com/sms/1/text/advanced");
     httpWebRequest.Method = "POST";
     httpWebRequest.ContentType = "application/json";
     httpWebRequest.Accept = "application/json";
     httpWebRequest.Headers["authorization"] = "Basic " + encryptedCredentials;

     List<Destination> destinations = new List<Destination>();
     foreach (var number in destinationAddress)
         destinations.Add(new Destination() { to = number });

     Message textMessage = new Message()
     {
         from = username,
         destinations = destinations,
         sendAt = schedule,
         text = message
     };

     List<Message> messages = new List<Message>();
     messages.Add(textMessage);

     //JSON POST REQUEST

     JsonRequest jsonRequest = new JsonRequest() { messages = messages };
     string jsonString = JsonHelper.JsonSerializer<JsonRequest>(jsonRequest);

     using (var streamWriter = new StreamWriter(httpWebRequest.GetRequestStream()))
     {
         streamWriter.Write(jsonString);
         streamWriter.Flush();
         streamWriter.Close();
     }

     //JSON GET RESPONSE

     var httpResponse = (HttpWebResponse)httpWebRequest.GetResponse();
     using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
     {
         var result = streamReader.ReadToEnd();
         var response = JsonHelper.JsonDeserialize<JsonResponse>(result);
     }
