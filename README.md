
import java.sql.*;
import java.util.*; 

public class FullProject {
	/*
	 * IMPORTANT NOTE:
	 * FIRST: Make sure that you are using the right port, database, and adminPassword
	 * SECOND: Make sure that you have the correct Tables in your database (I sent an email with all the tables, names, and items)
	 * THIRD: Make sure this file is in the correct folder on your computer
	 * 
	 */
	
		static String port = "localhost"; // What IP you are using
		static String database = "usersdb";
		static String adminPassword = "12345";

		public static void main(String[] args) throws Exception {
			loginView();
		}

		public static void loginView() {
			Scanner scnr = new Scanner(System.in);
			System.out.println("Welcome!\n");
			System.out.println("Select from one of the following options:");
			System.out.println("(R)egister, (L)ogin, (Q)uit\n");

			System.out.print("-");
			char input = scnr.nextLine().toLowerCase().charAt(0);

			String name;

			// LOOP till quits
			while (input != 'q') {
				System.out.println();
				// If register and correct user name go to main view
				if (input == 'r') {
					try {
						name = register(scnr);
						mainView(name, scnr);
					} catch (Exception e) {
						System.out.println("ERROR: Username not Valid\n");
					}
					// If Login correctly go to main view
				} else if (input == 'l') {
					try {
						name = login(scnr);
						mainView(name, scnr);
					} catch (Exception e) {
						System.out.println("ERROR: Credentials not Valid\n");
					}
					// Anything else
				} else {
					System.out.println("Invalid input\n");
				}

				System.out.println("Select from one of the following options:");
				System.out.println("(R)egister, (L)ogin, (Q)uit\n");
				System.out.print("-");
				input = scnr.nextLine().toLowerCase().charAt(0);
			}

		}

		public static boolean validName(Connection c, String name) {
			try {
				boolean ans = true;
				Statement stmt = c.createStatement();
				ResultSet rs = stmt.executeQuery("SELECT username FROM LOGINS;");

				while (rs.next()) {
					if (name.equals(rs.getString("username"))) {
						ans = false;
						break;
					}
				}

				rs.close();
				stmt.close();
				return ans;

			} catch (Exception e) {
				e.printStackTrace();
				System.err.println(e.getClass().getName() + ": " + e.getMessage());
				System.exit(0);
				return false;
			}
		}

		public static String register(Scanner scnr) throws Exception {
			// Get a valid user name
			System.out.print("Enter Username: ");
			String username = scnr.nextLine();

			Connection c = null;
			Statement stmt = null;
			String password;

			// Connect to the database
			Class.forName("org.postgresql.Driver");
			c = DriverManager.getConnection(
					"jdbc:postgresql://" + port + ":5432/" + database,
					"postgres", adminPassword);

			// Get a password
			System.out.print("Enter Password: ");
			password = scnr.nextLine();
			System.out.println();

			if (!validName(c, username)) {
				throw new Exception();
			}

			// Add username password to logins
			stmt = c.createStatement();
			String sql = "INSERT INTO LOGINS (username, password) "
					+ "VALUES('" + username + "', '" + password + "')";
			stmt.execute(sql);
			stmt.close();

			systemLogin(c, username);
			System.out.println("Welcome " + username + "!");
			c.close();

			return username;
		}

		public static String login(Scanner scnr) throws Exception {
			String username, password;
			// Get a valid user name
			System.out.print("Enter Username: ");
			username = scnr.nextLine();

			// Get connection
			Class.forName("org.postgresql.Driver");
			Connection c = DriverManager.getConnection(
					"jdbc:postgresql://" + port + ":5432/" + database,
					"postgres", adminPassword);

			// Get a password
			System.out.print("Enter Password: ");
			password = scnr.nextLine();
			System.out.println();

			// Get login tabble
			Statement stmt = c.createStatement();
			ResultSet rs = stmt.executeQuery("SELECT * FROM logins;");

			while (rs.next()) {
				if (username.equals(rs.getString("username")) && password.equals(rs.getString("password"))) {
					systemLogin(c, username);
					System.out.println("Welcome " + username + "!");

					rs.close();
					stmt.close();
					c.close();

					return username;
				}
			}
			rs.close();
			stmt.close();
			c.close();

			throw new Exception();
		}

		private static void systemLogin(Connection c, String username) throws Exception {
			Statement stmt = null;

			Class.forName("org.postgresql.Driver");
			c = DriverManager.getConnection(
					"jdbc:postgresql://" + port + ":5432/" + database,
					"postgres", adminPassword);

			stmt = c.createStatement();
			String sql = "INSERT INTO ACTIVE (username) "
					+ "VALUES('" + username + "')";
			stmt.execute(sql);
			stmt.close();
		}

		public static void mainView(String username, Scanner scnr) {
			String chatroomName;
			// initial output options
			System.out.println("Please select from the following options: ");
			System.out.println("(J)oin, (C)reate, (A)ccount, (L)ogout");
			System.out.print("\n-");
			char command = scnr.nextLine().toLowerCase().charAt(0);

			// LOOP till command is l
			while (command != 'l') {

				if (command == 'j') {
					System.out.println("\nPlease input chatroom name to join: ");
					chatroomName = scnr.nextLine();
					joinChatroom(chatroomName, username, scnr);

				} else if (command == 'c') {
					System.out.println("\nPlease input chatroom name to create: ");
					chatroomName = scnr.nextLine();
					createChatroom(chatroomName, username, scnr);

				} else if (command == 'a') {
					username = updateAcct(username, scnr);

				} else {
					System.out.println("ERROR: Invalid Input\n");
				}
				System.out.println("Please select from the following options: ");
				System.out.println("(J)oin, (C)reate, (A)ccount, (L)ogout");
				System.out.print("\n-");
				command = scnr.nextLine().toLowerCase().charAt(0);
			}
			logout(username, scnr);
		}

		public static String updateAcct(String username, Scanner scnr) {
			Connection c = null;
			Statement stmt = null;

			try {
				Class.forName("org.postgresql.Driver");
				c = DriverManager.getConnection(
						"jdbc:postgresql://" + port + ":5432/" + database,
						"postgres", adminPassword);
				c.setAutoCommit(false);

				// Get usernaem and password
				System.out.print("\nEnter new username: ");
				String newName = scnr.nextLine();
				System.out.print("\nEnter new Password: ");
				String newPassword = scnr.nextLine();

				// Check if vaild
				if (!validName(c, newName)) {
					System.out.println("ERROR: Username is Invalid!\n");
					return username;
				}

				// ADD new creds to LOGINS
				stmt = c.createStatement();
				stmt.executeUpdate("INSERT INTO LOGINS (username, password) "
						+ "VALUES('" + newName + "', '" + newPassword + "')");
				stmt.close();
				c.commit();

				// Delete old creds from Logins
				stmt = c.createStatement();
				stmt.executeUpdate("DELETE from LOGINS where username = '" + username + "';");
				stmt.close();
				c.commit();

				// Login new creds
				systemLogin(c, newName);

				// Logout old creds
				logout(username, scnr);

				c.close();
				return newName;

			} catch (Exception e) {
				e.printStackTrace();
				System.err.println(e.getClass().getName() + ": " + e.getMessage());
				System.exit(0);
				return username;
			}
		}

		public static void logout(String username, Scanner scnr) {
			Connection c = null;
			Statement stmt = null;

			try {
				Class.forName("org.postgresql.Driver");
				c = DriverManager.getConnection(
						"jdbc:postgresql://" + port + ":5432/" + database,
						"postgres", adminPassword);

				stmt = c.createStatement();
				stmt.executeUpdate("DELETE from ACTIVE where username = '" + username + "';");
				stmt.close();
				c.close();

			} catch (Exception e) {
				e.printStackTrace();
				System.err.println(e.getClass().getName() + ": " + e.getMessage());
				System.exit(0);
			}
		}

		public static void joinChatroom(String chatroomName, String username, Scanner scnr) {
			Connection c = null;
			Statement stmt = null;
			boolean found = false;

			try {
				// connect to database
				Class.forName("org.postgresql.Driver");
				c = DriverManager.getConnection(
						"jdbc:postgresql://" + port + ":5432/" + database,
						"postgres", adminPassword);

				// check that room name exists
				stmt = c.createStatement();
				ResultSet rs = stmt.executeQuery("SELECT * FROM CHATROOM");
				while (rs.next()) {
					// If chatroom name is found
					if (chatroomName.equals(rs.getString("NAME"))) {
						found = true;
						break;
					}
				}
				stmt.close();
				rs.close();
				c.close();

				if (!found) {
					System.out.println("\nERROR: Chatroom not found");
					return;
				}

				System.out.println("Joined \"" + chatroomName + "\"");
				roomView(chatroomName, username, scnr);

			} catch (Exception e) {
				e.printStackTrace();
				System.err.println(e.getClass().getName() + ": " + e.getMessage());
				System.exit(0);
			}
		}

		public static void createChatroom(String chatroomName, String username, Scanner scnr) {
			Connection c = null;
			Statement stmt = null;

			// Check that name is valid
			for (int i = 0; i < chatroomName.length(); ++i) {
				if (!Character.isLowerCase(chatroomName.charAt(i)) && !Character.isDigit(chatroomName.charAt(i))) {
					System.out.println("\nERROR: Invaild chatroom name (lowercase and numbers only)");
					return;
				}
			}

			try {
				// connect to database
				Class.forName("org.postgresql.Driver");
				c = DriverManager.getConnection(
						"jdbc:postgresql://" + port + ":5432/" + database,
						"postgres", adminPassword);

				// Insert new chat room into table
				stmt = c.createStatement();
				String sql = "INSERT INTO CHATROOM (name) "
						+ "VALUES ('" + chatroomName + "')";
				stmt.executeUpdate(sql);

				// New chat room table
				sql = "CREATE TABLE " + chatroomName
						+ "(ID INT PRIMARY KEY NOT NULL,"
						+ "USERNAME TEXT NOT NULL,"
						+ "CHAT TEXT NOT NULL)";
				stmt.executeUpdate(sql);

				// close and finish
				stmt.close();
				c.close();
				System.out.println("Chatroom \"" + chatroomName + "\" Created");
				joinChatroom(chatroomName, username, scnr);

			} catch (Exception e) {
				e.printStackTrace();
				System.err.println(e.getClass().getName() + ": " + e.getMessage());
				System.exit(0);
			}
		}

		public static int printNewChats(String chatroomName, int currChat) {
			Statement stmt;
			Connection c;

			try {
				Class.forName("org.postgresql.Driver");
				c = DriverManager.getConnection(
						"jdbc:postgresql://" + port + ":5432/" + database,
						"postgres", adminPassword);

				stmt = c.createStatement();
				ResultSet rs = stmt.executeQuery("SELECT * FROM " + chatroomName + " WHERE ID > " + currChat);
				while (rs.next()) {
					String chatName = rs.getString("username");
					String chatBody = rs.getString("chat");
					currChat = rs.getInt("ID");

					System.out.println(chatName + ": " + chatBody);
				}

				rs.close();
				stmt.close();
				c.close();

				return currChat;

			} catch (Exception e) {
				e.printStackTrace();
				System.err.println(e.getClass().getName() + ": " + e.getMessage());
				System.exit(0);
				return 0;
			}
		}

		public static void readIntput(String chatroomName, String username, String input, int currChat) {
			Statement stmt;
			Connection c;

			// IF command run Command method
			if (input.charAt(0) == '/') {
				runCommand(chatroomName, username, input);
				return;
			}

			// If not add text to table
			try {
				Class.forName("org.postgresql.Driver");
				c = DriverManager.getConnection(
						"jdbc:postgresql://" + port + ":5432/" + database,
						"postgres", adminPassword);

				stmt = c.createStatement();
				String sql = "INSERT INTO " + chatroomName + " (id, username, chat) " +
						"VALUES (" + currChat + ", '" + username + "', '" + input + "')";
				stmt.executeUpdate(sql);

				stmt.close();
				c.close();
				return;

			} catch (Exception e) {
				e.printStackTrace();
				System.err.println(e.getClass().getName() + ": " + e.getMessage());
				System.exit(0);
				return;
			}

		}

		public static void runCommand(String chatroomName, String username, String input) {
			Statement stmt;
			Connection c;

			try {
				Class.forName("org.postgresql.Driver");
				c = DriverManager.getConnection(
						"jdbc:postgresql://" + port + ":5432/" + database,
						"postgres", adminPassword);

				// IF /list print out all the active users
				if ("/list".equals(input)) {
					System.out.println("\nHere are the current users logged into this chat room: ");

					stmt = c.createStatement();
					ResultSet rs = stmt.executeQuery("SELECT * FROM ACTIVE;");
					while (rs.next()) { // checks if rs.next has a value
						String users = rs.getString("username"); // grabs the user names of everyone logged in the
																	// chat room
						System.out.println("USERNAME: " + users);
					}

				} // IF /leave leaves the chat room
				else if ("/leave".equals(input)) {
					System.out.println("\nYou have left the chat room.");
					inRoom = false;

				} // IF /history prints chat history
				else if ("/history".equals(input)) {
					System.out.println("\nHere are all of the past messages sent in this chat room: ");
					printNewChats(chatroomName, 0);

				} // IF /help prints commands
				else if ("/help".equals(input)) {
					System.out.println("\n/list shows all users currently in the room.");
					System.out.println("/leave allows you to leave the chat room.");
					System.out.println("/history displays all past messages for the room.");
					System.out.println("/help displays all available commands.");

				} else {
					System.out.println("\nNot a vaild command. Try /help.");
				}

			} catch (Exception e) {
				e.printStackTrace();
				System.err.println(e.getClass().getName() + ": " + e.getMessage());
				System.exit(0);
			}
		}

		static boolean inRoom;
		static int currChat;

		public static void roomView(String chatroomName, String username, Scanner scnr) {
			Connection c = null;
			Statement stmt = null;
			inRoom = true;

			// THIS WHOLE Try block is to find currChat ID
			// This is probs a better way to do this but idk
			try {
				Class.forName("org.postgresql.Driver");
				c = DriverManager.getConnection(
						"jdbc:postgresql://" + port + ":5432/" + database,
						"postgres", adminPassword);

				stmt = c.createStatement();
				ResultSet rs = stmt.executeQuery("SELECT * FROM " + chatroomName);
				while (rs.next()) {
					currChat = rs.getInt("ID");
				}

			} catch (Exception e) {
				e.printStackTrace();
				System.err.println(e.getClass().getName() + ": " + e.getMessage());
				System.exit(0);
			}

			// CREATE TREAD
			// It is kinda like a seperate main that runs at the same time
			Thread inputThread = new Thread(new Runnable() {
				@Override
				public void run() {
					while (inRoom) {
						if (scnr.hasNextLine()) {
							String input = scnr.nextLine();
							readIntput(chatroomName, username, input, currChat + 1);
						}
					}
				}
			});

			// Start running the tread
			inputThread.start();
			//
			while (inRoom) {
				currChat = printNewChats(chatroomName, currChat);
			}

		}
		}


			}

		}
		}

