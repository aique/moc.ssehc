```
openapi: 3.0.3
info:
  title: Notification System API
  description: Manages user notification options and sends notifications.
  version: 1.0.0
paths:
  /preferences/{username}:
    get:
      summary: Gets the user notification preferences to allow the user to review their values.
      description: Retrieves the user’s notification preferences for review. This can be handled by the reverse proxy module if the user has made this request before and the response is still in the reverse proxy cache. Otherwise, the request managers will handle the request, obtaining the response from the in-memory storage system and, if not found there, from the disk storage system. The username can be used to link the request with the preferences.
      parameters:
        - name: username
          in: path
          required: true
          description: The username for which to retrieve preferences.
          schema:
            type: string
        - name: verification_hash
          in: query
          required: true
          description: Hash used for verifying the authenticity of the request.
          schema:
            type: string
      responses:
        '200':
          description: User notification preferences successfully retrieved.
          content:
            application/json:
              schema:
                type: object
                properties:
                  notification_system:
                    type: string
                    enum: ['web', 'email', 'app']
                    description: The preferred notification method of the user.
              example:
                notification_system: 'email'
        '400':
          description: Bad Request - Missing or invalid verification hash or username.
        '500':
          description: Internal Server Error
    put:
      summary: Updates the user notification preferences.
      description: Modifies the user’s notification preferences with the provided values. This request will be handled by the request managers, who will modify the data in the disk storage system and update the in-memory storage system to delete or update the obsolete data.
      parameters:
        - name: username
          in: path
          required: true
          description: The username for which to update preferences.
          schema:
            type: string
        - name: verification_hash
          in: query
          required: true
          description: Hash used for verifying the authenticity of the request.
          schema:
            type: string
      requestBody:
        description: Notification preferences to update.
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                notification_system:
                  type: string
                  enum: ['web', 'email', 'app']
                  description: The preferred notification method.
              required:
                - notification_system
            example:
              notification_system: 'email'
      responses:
        '204':
          description: Event successfully created, no content to return.
        '400':
          description: Bad Request - Missing or invalid verification hash or invalid input.
        '500':
          description: Internal Server Error
  /new-event:
    post:
      summary: Creates a new event with the specified type.
      description: Allows the creation of a new event by specifying the event type and event name, and optionally the winner or agent. This request will be handled by the request managers, who will create a new message in the message queue module. This message will contain the necessary data to display the event description to the user and the channel through which it will be sent by the cloud messaging system. Optionally, this data can be stored in the database if, in the future, an event history is needed. The messages will be processed by the notification workers, who will create a batch of notifications to be sent to the cloud messaging system. After send it and get a successful response, they will delete the messages in the message queue.
      parameters:
        - name: verification_hash
          in: query
          required: true
          description: Hash used for verifying the authenticity of the request.
          schema:
            type: string
      requestBody:
        description: Details of the event to be created.
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                event_name:
                  type: string
                  description: The name of the event following the format 'fighter_1_name vs fighter_2_name'.
                  example: 'John Doe vs Jane Doe'
                event_type:
                  type: string
                  enum: ['major-piece-captured', 'check', 'check-mate', 'power-punch-landed', 'ko', 'tko']
                  description: The type of the event being created.
                winner:
                  type: string
                  description: The name of the winner. Required for events 'ko', 'tko', and 'check-mate'.
                  example: 'John Doe'
                agent:
                  type: string
                  description: The name of the person performing the action. Required for events 'check' and 'major-piece-captured'.
                  example: 'Alice Smith'
              required:
                - event_type
              oneOf:
                - properties:
                    event_type:
                      enum: ['ko', 'tko', 'check-mate']
                    winner:
                      type: string
                    event_name:
                      type: string
                  required:
                    - winner
                    - event_name
                - properties:
                    event_type:
                      enum: ['power-punch-landed', 'check', 'major-piece-captured']
                    agent:
                      type: string
                    event_name:
                      type: string
                  required:
                    - agent
                    - event_name
      responses:
        '204':
          description: Event successfully created, no content to return.
        '400':
          description: Bad Request - Missing or invalid verification hash, invalid input or missing properties.
        '500':
          description: Internal Server Error
```
