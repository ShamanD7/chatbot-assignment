import React, { useState, useEffect } from "react";
import { nhost } from "../nhost";
import { request, gql } from "graphql-request";

const GRAPHQL_ENDPOINT = "https://krqcuxnheobogsjouguw.hasura.ap-south-1.nhost.run/v1/graphqll"; // Replace with your Nhost backend URL

export default function Chats() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);

  // Fetch messages from Hasura
  const fetchMessages = async () => {
    const query = gql`
      query {
        messages(order_by: { created_at: asc }) {
          id
          content
          created_at
        }
      }
    `;
    try {
      const data = await request(GRAPHQL_ENDPOINT, query, {}, {
        Authorization: `Bearer ${nhost.auth.getJWTToken()}`
      });
      setMessages(data.messages);
    } catch (error) {
      console.error("Error fetching messages:", error);
    }
  };

  useEffect(() => {
    fetchMessages();
    // Optional: Poll every 3 seconds for new messages
    const interval = setInterval(fetchMessages, 3000);
    return () => clearInterval(interval);
  }, []);

  // Send message to Hasura
  const sendMessage = async (e) => {
    e.preventDefault();
    if (!input.trim()) return;

    setLoading(true);
    const mutation = gql`
      mutation ($content: String!) {
        insert_messages_one(object: { content: $content }) {
          id
          content
        }
      }
    `;
    try {
      await request(GRAPHQL_ENDPOINT, mutation, { content: input }, {
        Authorization: `Bearer ${nhost.auth.getJWTToken()}`
      });
      setInput("");
      fetchMessages(); // Refresh messages
    } catch (error) {
      console.error("Error sending message:", error);
    }
    setLoading(false);
  };

  return (
    <div className="chat-container">
      <h2>Chat Room</h2>
      <div className="chat-box" style={{ border: "1px solid #ccc", padding: "10px", height: "300px", overflowY: "auto" }}>
        {messages.map(msg => (
          <div key={msg.id} style={{ margin: "5px 0" }}>
            <span>{msg.content}</span>
          </div>
        ))}
      </div>
      <form onSubmit={sendMessage} style={{ marginTop: "10px" }}>
        <input
          type="text"
          value={input}
          onChange={e => setInput(e.target.value)}
          placeholder="Type your message..."
          style={{ width: "80%", padding: "5px" }}
        />
        <button type="submit" disabled={loading} style={{ padding: "5px 10px", marginLeft: "5px" }}>
          {loading ? "Sending..." : "Send"}
        </button>
      </form>
    </div>
  );
}
