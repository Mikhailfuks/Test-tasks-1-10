tasks 1 
public class MinValueUtil {
  public int findMin(int[] array) {
      int min = Integer.MAX_VALUE;
      for (int num : array) {
          if (num < min) {
              min = num;
          }
      }
      return min;
  }
}

import static org.junit.jupiter.api.Assertions.assertEquals;
import org.junit.jupiter.api.Test;
import java.util.Date;

public class DateUtilTest {
    @Test
    public void testFormat() {
        DateUtil dateUtil = new DateUtil();
        assertEquals("2023-08-28", dateUtil.format(new Date(1693248000000L))); // timestamp for 2023-08-28
    }
}

tasks 2
  package jcg.zheng.demo.service;

      import static org.junit.Assert.assertEquals;
      import static org.junit.Assert.assertNotNull;
      
      import org.junit.Test;
      
      import jcg.zheng.demo.entity.Person;
      
      public class TransformServiceTest {
      
        private TransformService testClass = new TransformService() ;
      
        @Test
        public void test_toDomain() {
          Person person = new Person();
          person.setCompanyName("test company");
          person.setfName("Mary");
          person.setlName("Zheng");
          person.setmName("shan");
          person.setPersonId(1);
          User user = testClass.toUserDomain(person);
      
          assertNotNull(user);
          assertEquals("test company", user.getCompanyName());
          assertEquals("Mary", user.getFirstName());
          assertEquals("Zheng", user.getLastName());
          assertEquals(1, user.getUserId().intValue());
        }
      
        @Test
        public void test_toEntity() {
          User user = new User();
      
          user.setCompanyName("test company");
          user.setFirstName("Mary");
          user.setLastName("Zheng");
          user.setUserId(Integer.valueOf(1));
      
          Person person = testClass.toUserEntity(user);
      
          assertNotNull(user);
          assertEquals("test company", person.getCompanyName());
          assertEquals("Mary", person.getfName());
          assertEquals("Zheng", person.getlName());
          assertEquals(1, person.getPersonId());
        }
      
      }
tasks 3 
      package jcg.zheng.demo.repository;

      import static org.junit.Assert.assertEquals;
      import static org.junit.Assert.assertNotNull;
      import static org.junit.Assert.assertNull;
      import static org.junit.Assert.assertTrue;
      
      import java.util.List;
      
      import org.junit.Before;
      import org.junit.Rule;
      import org.junit.Test;
      import org.junit.rules.Timeout;
      import org.junit.runner.RunWith;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
      import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
      import org.springframework.test.context.junit4.SpringRunner;
      
      import jcg.zheng.demo.entity.Person;
      
      @RunWith(SpringRunner.class)
      @DataJpaTest
      public class PersonRepositoryTest {
      
        @Rule
        public Timeout appTimeout = Timeout.millis(2000);
      
        @Autowired
        private TestEntityManager entityManager;
        @Autowired
        private PersonRepository personDao;
      
        @Before
        public void setup() {
          assertNotNull(entityManager);
          assertNotNull(personDao);
      
          // prepare two persons
          Person mary = new Person();
          mary.setfName("Mary");
          mary.setCompanyName("Test");
          entityManager.persist(mary);
      
          Person alex = new Person();
          alex.setfName("Alex");
          alex.setCompanyName("Alex company");
          entityManager.persist(alex);
      
        }
      
        @Test
        public void findAll_return_list_when_found() {
          List<Person> found = personDao.findAll();
      
          assertNotNull(found);
          assertEquals(2, found.size());
        }
      
        @Test
        public void findByCompany_return_person_when_found() {
          List<Person> found = personDao.findByCompany("Test");
      
          assertNotNull(found);
          assertEquals("Mary", found.get(0).getfName());
        }
      
        @Test
        public void findByCompany_return_emptylist_when_not_found() {
          List<Person> found = personDao.findByCompany("Test-notExist");
      
          assertNotNull(found);
          assertTrue(found.isEmpty());
      
        }
      
        @Test
        public void findOne_return_null_when_not_found() {
          Person found = personDao.findOne(-9);
      
          assertNull(found);
        }
      
      }

tasks 4 
package jcg.zheng.demo.service;

      import org.springframework.stereotype.Component;
      
      import jcg.zheng.demo.entity.Person;
      
      @Component
      public class TransformService {
      
        public User toUserDomain(final Person person) {
          User user = new User();
          user.setCompanyName(person.getCompanyName());
          user.setFirstName(person.getfName());
          user.setLastName(person.getlName());
          user.setUserId(person.getPersonId());
          return user;
        }
      
        public Person toUserEntity(final User user) {
          Person person = new Person();
          person.setCompanyName(user.getCompanyName());
          person.setfName(user.getFirstName());
          person.setlName(user.getLastName());
          if (user.getUserId() != null) {
            person.setPersonId(user.getUserId());
          }
          return person;
        }
      }

tasks 5 
 package jcg.zheng.demo.service;

      import java.util.ArrayList;
      import java.util.List;
      
      import javax.transaction.Transactional;
      
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.stereotype.Component;
      
      import jcg.zheng.demo.entity.Person;
      import jcg.zheng.demo.exception.UserNotFoundException;
      import jcg.zheng.demo.repository.PersonRepository;
      
      @Component
      @Transactional
      public class UserServiceImpl implements UserService {
      
        @Autowired
        private PersonRepository personDao;
      
        @Autowired
        private TransformService transformer;
      
        @Override
        public void deleteById(Integer personId) {
          personDao.delete(personId);
        }
      
        @Override
        public User findById(Integer personId) {
          Person found = personDao.findOne(personId);
      
          if (found == null) {
            throw new UserNotFoundException("not found user", personId);
          }
          return transformer.toUserDomain(found);
        }
      
        @Override
        public User save(User user) {
          Person saved = personDao.save(transformer.toUserEntity(user));
          return transformer.toUserDomain(saved);
        }
      
        @Override
        public List<User> searchByCompanyName(String companyName) {
          List<Person> persons = personDao.findByCompany(companyName);
          List<User> users = new ArrayList<>();
          for (Person person : persons) {
            users.add(transformer.toUserDomain(person));
          }
          return users;
        }
      }

tasks 6
 package jcg.zheng.demo.entity;

      import javax.persistence.Entity;
      import javax.persistence.GeneratedValue;
      import javax.persistence.GenerationType;
      import javax.persistence.Id;
      
      @Entity
      public class Person {
      
        private String companyName;
      
        private String fName;
        private String lName;
        private String mName;
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private int personId;
      
        public String getCompanyName() {
          return companyName;
        }
      
        public String getfName() {
          return fName;
        }
      
        public String getlName() {
          return lName;
        }
      
        public String getmName() {
          return mName;
        }
      
        public int getPersonId() {
          return personId;
        }
      
        public void setCompanyName(String companyName) {
          this.companyName = companyName;
        }
      
        public void setfName(String fName) {
          this.fName = fName;
        }
      
        public void setlName(String lName) {
          this.lName = lName;
        }
      
        public void setmName(String mName) {
          this.mName = mName;
        }
      
        public void setPersonId(int personId) {
          this.personId = personId;
        }
      
      }


tasks 7 
              public int assignPermission() {
                if(user.getRole().equals("admin")) {
                    String username = user.getUsername();
                    System.out.println("Assign special permissions for user " + username);
                    return 1;
                } else {
                    System.out.println("Cannot assign permission");
                    return -1;
                }
            }

tasks 8 
  public void sortArray(int[] array) {
                  int n = array.length;
                  for (int i = 0; i < n-1; i++) {
                      for (int j = 0; j < n-i-1; j++) {
                          if (array[j] > array[j+1]) {
                              int temp = array[j];
                              array[j] = array[j+1];
                              array[j+1] = temp;
                          }
                      }
                  }
              }



































































































































      
