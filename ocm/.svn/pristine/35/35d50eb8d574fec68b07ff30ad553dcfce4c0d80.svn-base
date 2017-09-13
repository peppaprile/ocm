/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package ocm;

import java.io.IOException;
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.Collection;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Timer;
import java.util.TimerTask;
import java.util.concurrent.TimeoutException;
import java.util.logging.Level;
import java.util.logging.Logger;
import javax.xml.soap.MessageFactory;
import javax.xml.soap.MimeHeaders;
import javax.xml.soap.SOAPBody;
import javax.xml.soap.SOAPConnection;
import javax.xml.soap.SOAPConnectionFactory;
import javax.xml.soap.SOAPElement;
import javax.xml.soap.SOAPEnvelope;
import javax.xml.soap.SOAPMessage;
import javax.xml.soap.SOAPPart;
import org.jivesoftware.smack.ConnectionConfiguration;
import org.jivesoftware.smack.PacketListener;
import org.jivesoftware.smack.Roster;
import org.jivesoftware.smack.RosterEntry;
import org.jivesoftware.smack.RosterGroup;
import org.jivesoftware.smack.RosterListener;
import org.jivesoftware.smack.XMPPConnection;
import org.jivesoftware.smack.XMPPException;
import org.jivesoftware.smack.packet.Message;
import org.jivesoftware.smack.packet.Packet;
import org.jivesoftware.smack.packet.Presence;
import org.jivesoftware.smack.packet.RosterPacket;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;


/**
 *
 * @author gaprile
 */
public class Ocm { //implements PacketListener {
  private static String SmackSvrIP="";

   private static String SmackSvrName="";
   
   static  Timer timer;
   static  Timer timerT;
    boolean isDebugOn=true;
   
     
Roster roster;
Collection<RosterEntry> entries;
RosterListener rl;

private HashMap PresenceList = new HashMap<String, String>();

private final int SmackSvrport= 5222;

XMPPConnection connection;


Collection<RosterGroup>  groups;

 

/*

    @Override
    public void processPacket(Packet packet) 
    {
        
        if(packet.getClass() == Presence.class)
        {
            Presence p = (Presence) packet;
            
            groups = roster.getGroups();

            for(RosterGroup g:groups)
            {
                entries  = g.getEntries();
            } 
        
        }   
        if(packet.getClass() == RosterPacket.class)
        {
            RosterPacket rp = (RosterPacket) packet;
          
            groups = roster.getGroups();

   
            for(RosterGroup g:groups)
            {
                entries  = g.getEntries();
            } 
            
         
        }   
        else
        {
            
       
            Message message = (Message) packet;


        }
      
       
       
        
    }
   */ 
       
   class sendAlertMsgIfUnavalaible extends TimerTask 
    {
        String to, what;
        public void run() 
        {
               String timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                       
                if(PresenceList.get(to)!=null){
                    String p = PresenceList.get(to).toString();
                    if(p.contentEquals("available")){
                        System.out.println(timeStamp + " send alert at app time because not avalaible 5 min before"+this.to);

                        Message tmpMessage   = new Message();
                        tmpMessage.setBody(this.what);
                        Packet packet = tmpMessage;
                        packet.setFrom("ocm@"+SmackSvrName);
                        packet.setTo(this.to);
                        connection.sendPacket(packet);

                    }
                }
                
              
        }
        
        
   
    }
    class sendAlertMsg extends TimerTask 
    {
        String to, what;
        
        public void run() 
        {                 
            
            if(PresenceList.get(to)!=null){
                
                 String timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                       
                String p =  PresenceList.get(to).toString();
                System.out.println(timeStamp+" send alert 5 min before"+this.to); 
                if(p.contentEquals("available")){
                    

                    Message tmpMessage   = new Message();
                    tmpMessage.setBody(this.what);
                    Packet packet = tmpMessage;
                    packet.setFrom("ocm@"+SmackSvrName);
                    packet.setTo(this.to);
                    
                    if(isDebugOn)
                    {
                        timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                        System.out.println( timeStamp +" Send alert as agent avalaible, 5 min before appointment to "+this.to + " content:"+ this.what );
                    }
                    
                    connection.sendPacket(packet);     
                  
                }else{
                  //  System.out.println("send alert but not avalible "+this.to); 
                    sendAlertMsgIfUnavalaible samiu = new sendAlertMsgIfUnavalaible();
                   
                    samiu.to = this.to;
                    samiu.what= this.what;
                    
                    if(isDebugOn)
                    {
                        timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                        System.out.println(timeStamp +" Schedule alert as agent NOT avalaible in 5 min, appointment to "+this.to + " content:"+ this.what );
                    }
                    
                    Timer timer3 = new Timer();

                    timer3.schedule(samiu, 300000);//5minuti
  
          
                }
                
                
            }else{
                 String timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                       
                System.out.println(timeStamp +" send alert but not avalible "+this.to); 
                sendAlertMsgIfUnavalaible samiu = new sendAlertMsgIfUnavalaible();
                    
                    samiu.to = this.to;
                    samiu.what= this.what;
                    if(isDebugOn)
                    {
                        timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                        System.out.println(timeStamp +" Schedule alert as agent NOT in presence list in 5 min, appointment to "+this.to + " content:"+ this.what );
                    }
                    Timer timer3 = new Timer();

                    timer3.schedule(samiu, 300000);//5minuti
            }
            
          
        }   
    
  }  
    
   
   private static SOAPMessage createSOAPRequest(String serverName, String hour, String minutes) throws Exception {
        MessageFactory messageFactory = MessageFactory.newInstance();
        SOAPMessage soapMessage = messageFactory.createMessage();
        SOAPPart soapPart = soapMessage.getSOAPPart();

        String serverURI = "http://tempuri.org/";

        // SOAP Envelope
        SOAPEnvelope envelope = soapPart.getEnvelope();
        envelope.addNamespaceDeclaration("soap", serverURI);

       
        // SOAP Body
        SOAPBody soapBody = envelope.getBody();
        SOAPElement soapBodyElem = soapBody.addChildElement("AppointmentList_reminder", "soap");
        SOAPElement soapBodyElem1 = soapBodyElem.addChildElement("serverName", "soap");
        soapBodyElem1.addTextNode(serverName);
        SOAPElement soapBodyElem2 = soapBodyElem.addChildElement("hour", "soap");
        soapBodyElem2.addTextNode(hour);
        SOAPElement soapBodyElem3 = soapBodyElem.addChildElement("minutes", "soap");
        soapBodyElem3.addTextNode(minutes);
        SOAPElement soapBodyElem4 = soapBodyElem.addChildElement("code", "soap");
        soapBodyElem4.addTextNode("l44gg8Aj6C4=");


        MimeHeaders headers = soapMessage.getMimeHeaders();
        headers.addHeader("SOAPAction", serverURI  + "AppointmentList_reminder");

        soapMessage.saveChanges();

        /* Print the request message */
      //  System.out.print("Request SOAP Message = ");
      //  soapMessage.writeTo(System.out);
       // System.out.println();

        return soapMessage;
    }

    private static SOAPMessage createSOAPRequest2(String serverName, String hour) throws Exception {
        MessageFactory messageFactory = MessageFactory.newInstance();
        SOAPMessage soapMessage = messageFactory.createMessage();
        SOAPPart soapPart = soapMessage.getSOAPPart();

        String serverURI = "http://tempuri.org/";

        // SOAP Envelope
        SOAPEnvelope envelope = soapPart.getEnvelope();
        envelope.addNamespaceDeclaration("soap", serverURI);

       
        // SOAP Body
        SOAPBody soapBody = envelope.getBody();
        SOAPElement soapBodyElem = soapBody.addChildElement("HourlyCaseCheck", "soap");
        SOAPElement soapBodyElem1 = soapBodyElem.addChildElement("serverName", "soap");
        soapBodyElem1.addTextNode(serverName);
        SOAPElement soapBodyElem2 = soapBodyElem.addChildElement("hour", "soap");
        soapBodyElem2.addTextNode(hour);
    
        SOAPElement soapBodyElem4 = soapBodyElem.addChildElement("code", "soap");
        soapBodyElem4.addTextNode("l44gg8Aj6C4=");


        MimeHeaders headers = soapMessage.getMimeHeaders();
        headers.addHeader("SOAPAction", serverURI  + "HourlyCaseCheck");

        soapMessage.saveChanges();

        return soapMessage;
    }

    
    
    
    
    private void printSOAPResponse(SOAPMessage soapResponse) throws Exception {

        SOAPBody soapBody = soapResponse.getSOAPBody();
        for (Element response : elements(soapBody.getElementsByTagName("AppointmentList_reminderResponse"))) {
          for (Element result : elements(response.getElementsByTagName("AppointmentList_reminderResult"))) {
            List<Element> children = elements(result.getChildNodes());
            for(int i=0;i< children.size();i++){
                Element resultData = named(children.get(i), "OCM_Call");
                List<Element> resultDataChildren = elements(resultData.getChildNodes());
                String id = (named(resultDataChildren.get(0), "ID").getTextContent());
                String User_ID = (named(resultDataChildren.get(3), "User_ID").getTextContent());
                String User_Extension = (named(resultDataChildren.get(4), "User_Extension").getTextContent());
                String CustomerName = (named(resultDataChildren.get(5), "CustomerName").getTextContent());
                String Status = (named(resultDataChildren.get(6), "Status").getTextContent());
                String Appt = (named(resultDataChildren.get(7), "Appt").getTextContent());

                String[] splitted =  Appt.split("T");
                String orario= splitted[1];
                String[] splitted1 =  orario.split(":");
                String hh= splitted1[0];
                String mm= splitted1[1];
                int hhint = Integer.parseInt(hh);
                int mmint = Integer.parseInt(mm);


                sendAlertMsg sam= new sendAlertMsg();
                Calendar cal = Calendar.getInstance();
             
          
                int nowhour = cal.get(Calendar.HOUR_OF_DAY);
                int nowminutes = cal.get(Calendar.MINUTE);
                int nowseconds = cal.get(Calendar.SECOND);
                
                 String timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                       
                  System.out.println(timeStamp +" User form db: "+ User_Extension);
                
                
                
            
                //LocalDateTime now = LocalDateTime.now();

               // int nowhour = (cal.getHour());
               // int nowminutes = (now.getMinute());
               // int nowseconds = (now.getSecond());

                int n = nowseconds + nowminutes*60 + nowhour*60*60;
                int aptTime =  mmint*60 + hhint*60*60;


                    int anticipo=5;

                    int ts= ((aptTime-anticipo*60) - n) *1000 ;


                    if(ts >=0){

                        sam.to = User_Extension+"@"+SmackSvrName;
                        sam.what= "OCM:"+CustomerName+":"+id+":"+hh+":"+mm;
                        Timer timer1 = new Timer();

                        timer1.schedule(sam, ts);//
                       // timer.schedule(sam, 0, 10000 + i*1000);//10 sec
                    }                                            
                }

            }
        }
    }

    private static List<Element> elements(NodeList nodes) {
       List<Element> result = new ArrayList<Element>(nodes.getLength());
       for (int i = 0; i < nodes.getLength(); i++) {
         Node node = nodes.item(i);
         if (node instanceof Element)
           result.add((Element)node);
       }
       return result;
     }

     private static Element named(Element elem, String name) {
       if (!elem.getNodeName().equals(name))
         throw new IllegalArgumentException("Expected " + name + ", got " + elem.getNodeName());
       return elem;
     }
     
    class QueryWS2 extends TimerTask 
    {
        public void run() 
        {
           
           
  
            try {
            // Create SOAP Connection
            SOAPConnectionFactory soapConnectionFactory = SOAPConnectionFactory.newInstance();
            SOAPConnection soapConnection = soapConnectionFactory.createConnection();
             String url = "http://tloader.acer-euro.com/services/OCM.asmx";

          //  LocalDateTime now = LocalDateTime.now();
            
            Calendar cal = Calendar.getInstance();
             
           // System.out.println("QueryWS2! " + cal.toString()); 
            int nowhour = cal.get(Calendar.HOUR_OF_DAY);
             

        //    String hour = String.valueOf(now.getHour());
                String hour = String.valueOf(nowhour);

            SOAPMessage soapResponse = soapConnection.call(createSOAPRequest2( SmackSvrName,hour), url);

            soapConnection.close();
            } catch (Exception e) {
                System.err.println("Error occurred while sending SOAP2 Request to Server");
                e.printStackTrace();
            }
           
           
        }
    }
    class CheckList extends TimerTask 
    {
        public void run() 
        {
                     
            try {
            // Create SOAP Connection
            SOAPConnectionFactory soapConnectionFactory = SOAPConnectionFactory.newInstance();
            SOAPConnection soapConnection = soapConnectionFactory.createConnection();
            String url = "http://tloader.acer-euro.com/services/OCM.asmx";

         //   LocalDateTime now = LocalDateTime.now();

          //  String hour = String.valueOf(now.getHour());
           // String minutes = String.valueOf(now.getMinute());

         //   hour="12";
         //   minutes="50";
            
               Calendar cal = Calendar.getInstance();
              String timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                       
                System.out.println(timeStamp + " checklist!"); 
                int nowhour = cal.get(Calendar.HOUR_OF_DAY);
                int nowminutes = cal.get(Calendar.MINUTE);
             
                
            String hour = String.valueOf(nowhour);
           String minutes = String.valueOf(nowminutes);
            
      // System.out.println(hour +" "+ minutes);
            SOAPMessage soapResponse = soapConnection.call(createSOAPRequest( SmackSvrName,hour, minutes), url);

            // Process the SOAP Response
            printSOAPResponse(soapResponse);

            soapConnection.close();
            } catch (Exception e) {
                System.err.println("Error occurred while sending SOAP Request to Server");
                e.printStackTrace();
            }
        }   
    }

       
    public void login(String userName, String password) throws XMPPException
    {
        ConnectionConfiguration config = new ConnectionConfiguration(SmackSvrIP, SmackSvrport, SmackSvrName);
       
       
       
        connection = new XMPPConnection(config);
 
        connection.connect();
        connection.login(userName, password);
        try 
        {
            Thread.sleep(10000);
        } 
        catch (InterruptedException ex) 
        {
               System.out.println(ex);
        }
        
       // connection.addPacketListener(this, null);
      
               
      

     
    }
    public void sendMessage(String message, String to) throws XMPPException
    {       
        Message tmpMessage   = new Message();
        tmpMessage.setBody(message);
        Packet packet = tmpMessage;
        packet.setFrom("ocm@"+SmackSvrName);
        
        packet.setTo(to);
        connection.sendPacket(packet);      
    }
    
    public  Ocm() throws XMPPException
    {
        System.out.println("ocm version: ");
          this.login("ocm", "ocm");

    }
    

    public void run(RosterListener rl) throws IOException,  TimeoutException, InterruptedException,   XMPPException
    {   
       

        roster = connection.getRoster();   
        groups = roster.getGroups();
      
      
       

        for(RosterGroup g:groups)
        {
            entries  = g.getEntries();
            for(RosterEntry r:entries)
            {    
                PresenceList.put(r.getUser(), roster.getPresence(r.getUser()).getType());
               
            }
        }
        roster.addRosterListener(rl);
   

     System.out.println(PresenceList.size());
      CheckList cl= new CheckList();
        timer = new Timer();
        timer.schedule(cl, 0, 900000);//15 m

        QueryWS2 qws2= new QueryWS2();
        timerT = new Timer();
        timerT.schedule(qws2, 0, 3600000);//60 m

     //   Thread.sleep(259200000);  //3gg
    }
   
    public static void main(String[] args) throws XMPPException
    {
        
      try {
          SmackSvrIP=InetAddress.getLocalHost().getHostAddress();
           SmackSvrName=InetAddress.getLocalHost().getHostName();
      } catch (UnknownHostException ex) {
          Logger.getLogger(Ocm.class.getName()).log(Level.SEVERE, null, ex);
      }
           
            
      //      SmackSvrIP = "10.60.34.43";
      //     SmackSvrName = "itctapx1e.intra.acer-euro.com";
        final Ocm ocm = new Ocm();

        try
        {

          

           RosterListener rl;
            rl = new RosterListener() {
                
                @Override
                public void entriesDeleted(Collection<String> addresses) {
                    for (Iterator iterator = addresses.iterator(); iterator.hasNext();) {
                        String clctn0 = (String) iterator.next();
                        String tempa[] = clctn0.split("/");
                        String agent = tempa[0];
                        ocm.PresenceList.remove(agent);
                        String timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                         
                        System.out.println(timeStamp + " agent removed " + agent);
                    }
                }
                @Override
                public void entriesUpdated(Collection<String> addresses) {
                    for (Iterator iterator = addresses.iterator(); iterator.hasNext();) {
                        String clctn0 = (String) iterator.next();
                        String tempa[] = clctn0.split("/");
                        String agent = tempa[0];
                        ocm.PresenceList.put(agent, "undefined");
                        String timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                        
                        System.out.println(timeStamp + " agent updated " + agent);
                    } 
                }
                @Override
                public void entriesAdded(Collection<String> clctn) {
                    for (Iterator iterator = clctn.iterator(); iterator.hasNext();) {
                        String clctn0 = (String) iterator.next();
                        String tempa[] = clctn0.split("/");
                        String agent = tempa[0];
                        ocm.PresenceList.put(agent, "undefined");
                        String timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                        
                        System.out.println(timeStamp + " agent added " + agent);
                         
                    }
                    
                }
                @Override
                public void presenceChanged(Presence presence) {
                    
                    
                    String temp = presence.getFrom();
                    String tempa[] = temp.split("/");
                    String agent = tempa[0];
                    
                    ocm.PresenceList.put(agent, presence.getType());
                    String timeStamp = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Calendar.getInstance().getTime());
                    
                    System.out.println(timeStamp + " agent presence changed " + agent);
                    
                }
            };
            

         //   ocm = new Ocm();
            try 
            {
                ocm.run(rl);
            }
            catch (Exception ex) 
            {
               System.out.println(ex);
            }
        }
        catch (Exception ex)
        {
            System.out.println(ex);
        }

    }

    
}
