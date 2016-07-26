---
title: "使用Java发送SMTP邮件的类“SendMail”"
date: 2016-04-06 10:36:21 +0800
categories: [program]
tags: [java, smtp]
---

``` java
package org.vxwo.tools.simple;
 
import java.io.UnsupportedEncodingException;
import java.util.Properties;
 
import javax.mail.Message;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeMultipart;
import javax.mail.internet.MimeUtility;
 
public class SendMail {
  private boolean auth;
  private String host;
	private int port;
	private boolean useSSL;
	private String username;
	private String password;
 
	public SendMail(String host) {
		this(host, 25, false);
	}
 
	public SendMail(String host, int port, boolean useSSL) {
		auth = false;
		this.host = host;
		this.port = port;
		this.useSSL = useSSL;
	}
 
	public SendMail(String host, String username, String password) {
		this(host, 25, false, username, password);
	}
 
	public SendMail(String host, int port, boolean useSSL, String username,
			String password) {
		auth = true;
		this.host = host;
		this.port = port;
		this.useSSL = useSSL;
		this.username = username;
		this.password = password;
	}
 
	private Session getSession() {
		Properties props = new Properties();
		props.put("mail.smtp.host", host);
		props.put("mail.smtp.port", String.valueOf(port));
		props.put("mail.smtp.auth", String.valueOf(auth));
		if (useSSL) {
			props.put("mail.smtp.socketFactory.class",
					"javax.net.ssl.SSLSocketFactory");
			props
					.put("mail.smtp.socketFactory.fallback", String
							.valueOf(false));
			props.put("mail.smtp.socketFactory.port", String.valueOf(port));
		}
		return Session.getDefaultInstance(props);
	}
 
	private void transport(Session session, MimeMessage msg) throws Exception {
		Transport transport = session.getTransport("smtp");
		try {
			if (auth)
				transport.connect(username, password);
			else
				transport.connect();
			transport.sendMessage(msg, msg.getAllRecipients());
		} finally {
			transport.close();
		}
	}
 
	private InternetAddress encodeAddress(InternetAddress addr, String charset)
			throws UnsupportedEncodingException {
		String personal = addr.getPersonal();
		if (personal != null && personal.length() > 0)
			addr.setPersonal(MimeUtility.encodeText(personal, charset, "B"));
		return addr;
	}
 
	private InternetAddress[] encodeAddress(InternetAddress[] addrs,
			String charset) throws UnsupportedEncodingException {
		for (int i = 0; i < addrs.length; i++)
			addrs[i] = encodeAddress(addrs[i], charset);
		return addrs;
	}
 
	public void sendText(String from, String to, String subject, String text)
			throws Exception {
		sendText(from, to, subject, text, System.getProperty("file.encoding"));
	}
 
	public void sendText(String from, String to, String subject, String text,
			String charset) throws Exception {
		Session session = getSession();
		// create message
		MimeMessage msg = new MimeMessage(session);
		msg.setFrom(encodeAddress(new InternetAddress(from), charset));
		msg.setRecipients(Message.RecipientType.TO, encodeAddress(
				InternetAddress.parse(to), charset));
		msg.setSubject(MimeUtility.encodeText(subject, charset, "B"));
		MimeBodyPart mbp = new MimeBodyPart();
		mbp.setContent(text, "text/plain; charset=" + charset);
		mbp.setHeader("Content-Transfer-Encoding", "base64");
		MimeMultipart mp = new MimeMultipart();
		mp.addBodyPart(mbp);
		msg.setContent(mp);
		msg.saveChanges();
		// send message
		transport(session, msg);
	}
 
	public void sendHtml(String from, String to, String subject, String html)
			throws Exception {
		sendHtml(from, to, subject, html, System.getProperty("file.encoding"));
	}
 
	public void sendHtml(String from, String to, String subject, String html,
			String charset) throws Exception {
		Session session = getSession();
		// create message
		MimeMessage msg = new MimeMessage(session);
		msg.setFrom(encodeAddress(new InternetAddress(from), charset));
		msg.setRecipients(Message.RecipientType.TO, encodeAddress(
				InternetAddress.parse(to), charset));
		msg.setSubject(MimeUtility.encodeText(subject, charset, "B"));
		MimeBodyPart mbp = new MimeBodyPart();
		mbp.setContent(html, "text/html; charset=" + charset);
		mbp.setHeader("Content-Transfer-Encoding", "base64");
		MimeMultipart mp = new MimeMultipart();
		mp.addBodyPart(mbp);
		msg.setContent(mp);
		msg.saveChanges();
		// send message
		transport(session, msg);
	}
 
}
```

